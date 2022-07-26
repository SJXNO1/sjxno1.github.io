# yoyoSqlite.js 2.0.3

## 前言

随着LiteLoaderBDS新版本加载器的支持 已经有了sqlite3 数据库的方法,但是它提供的方法对于sql不熟悉的人不是很友好,而且复用并不是很方便,所以封装了这个链式方法库(可能有些地方写得不是很好)!但是我会尽力去完善它!

我是Yoyo感谢您的浏览!!!!

仓库地址:[https://gitee.com/Y_oyo/yoyo-mcbe-lite-xloader-item/tree/master/sql](https://gitee.com/Y_oyo/yoyo-mcbe-lite-xloader-item/tree/master/sql)

## 引入

```javascript
const { sqlite } = require('./yoyo-mcbe-lite-xloader-item/sql/yoyoSqlite-2.0.3');
```

## 连接sqlite数据库

```javascript
/* 多个参数的时候你可以传对象 */
var isSetConfig = sqlite.connectDb({
    path: '.\\plugins\\sql\\yoyo.db',
    create: true,/* 不存在是否自动创建 */
});
/* 或者你可以这样 */
var isSetConfig = sqlite.connectDb('.\\plugins\\sql\\yoyo.db');

if (isSetConfig) {
    log('数据库连接成功');
} else {
    log('数据库连接失败');
}
```

## 其它设置

```JavaScript
sqlite.table_prefix = 'yoyo_';//你可以设置一个表前缀(然后使用name直接用名即可) table它需要完整的名
sqlite.isShowSql = true;//你可以调试的时候显示sql语句
```

## 创建表

```javascript
let sql = sqlite.createTable('user',{
    id:{
        key:true,//是否为主键(每个表只有一个)
        auto:true//默认true 如果不需要自增可以设置false
    },
    account:{
        type:'integer',//字段类型
        default:'not null',//默认值是null
        unique:false//是否保持唯一值
    },
    name:{
        type:'varchar(15)'//字符串15位的
    },
    describe:'varchar(100)'//如果只有类型(默认null的字段)
});
log('表创建:',sql?'成功':'失败');
```
## 删除表 和 清空表(2.0.2)
```javascript
sqlite.delTable('user1');//删除表
sqlite.clearTable('user12');//清空表所有数据
let isTab = sqlite.isTable('mcuc');//判断指定表是否存在
```


## 添加数据

```javascript
var result = sqlite.table('user')//需要操作的完整表名
.field(['name','account','describe'])//设置字段
.insert(['我是名称',123456,'我是描述']);//添加一条数据
//你也可以添加多条(比如这样)
.insert([
    ['我是名称1',123456,'我是描述1'],
    ['我是名称2',123456]//(这里字段少了一个,我们不建议这样写 但是它会被null填充)
]);
//或许你想这样做(这样直接指定字段)
var result = sqlite.table('user')//需要操作的完整表名
.insert({name:'我是名称呀',account:9877,describe:'说得是这样?'});//添加一条数据
//任然,你也可以添加多条(比如这样)
.insert([
    {name:'我是名称呀',account:9877,describe:'说得是这样?'},
    {name:'我是名称呀',account:9877}//(这里字段少了一个,我们不建议这样写 但是它会被null填充)
]);
```

## 删除数据

```javascript
var result = sqlite.table('user')//需要操作的完整表名
.where('id',1)//获取你可以添加一些条件  (id = 1)
.where('name','<>','yoyo')//获取你可以添加一些条件  (name <> 'yoyo')
.where({
    account:12345,
    rmb:88
})//获取你可以添加一些条件  (account = 12345 AND rmb = 88)
.where([
    ['name','like','%yoyo%'],
    ['ps','in','(1,2,3,4)'],
])//获取你可以添加一些条件  (name like %yoyo% AND ps in (1,2,3,4))
.limit(5)//获取还可以限制删除的条数
.delete();
```

## 更新数据(修改)

```javascript
var result = sqlite.table('user')//需要操作的完整表名
.where('id',1)//获取你可以添加一些条件  (id = 1)
.update({
    rmb:99999,
    name:'我被修改了'
});
//修改id = 1 的 rmb 为 99999 ,name 为 我被修改了
```

## 查询数据

```javascript
!!!!!!!!!!!!!查询返回值有修改哦!!!!!看我
result.data;//查询到的数据
result.field;//字段
result.get(0,'name');//获取索引0的name字段数据 方便开发者快速获取
result.total;//查询到的数据总数量

var result = sqlite.table('user')//需要操作的完整表名
.where('id',1)//获取你可以添加一些条件  (id = 1)
.find();//查询一条数据

//或者
.select();//查询多条
//你任然可以用
.limit(5)
//or
.limit(0,18)
//去限制它
//我们看看排序
.order('id')//默认升序
.order('id','desc')//设置降序
.order({
    id:'asc',
    rmb:'desc'
})//这种方式 2.0.0 还未支持 可能在 2.0.0 +会添加此方式

//你也可以单独设置字段显示默认是 *
.field('id,name')//只要两个字段
.field('id,name AS 名称')//或者用来取名也可以

//多表联查也是支持的
var result = sqlite.table('user')//需要操作的完整表名
.alias('u')//取个别名
.join('user2 u2','u.id = u2.id')//你可以这样的当然它还有(leftJoin,rightJoin,fullJoin)他们分别代表(INNER JOIN,LEFT JOIN,RIGHT JOIN,FULL JOIN)

.whereRaw(sql,bind);//sql是字符串,bind 是 Number|String|Array
.whereRaw("id = 1 OR name = 'yoyo'");//这个是未加工的
.whereRaw("id = ? OR name = ?",[1,'yoyo']);//这个是未加工的


//快捷的聚合查询
//统计数量
var result = sqlite.table('user')//需要操作的完整表名
.count();//统计数量
.count('account');//依然也可以指定字符

//获取最大值
var result = sqlite.table('user').max('rmb');//获取指定字段的最大值(你必须给它指定字段)

//获取最小值
var result = sqlite.table('user').min('rmb');//获取指定字段的最小值(你必须给它指定字段)

//获取平均值
var result = sqlite.table('user').avg('rmb');//获取指定字段的平均值(你必须给它指定字段)

//获取总和
var result = sqlite.table('user').sum('rmb');//获取指定字段的总和(你必须给它指定字段)
```

## 分页查询

```javascript
var result = sqlite.table('user').pages(1,10);//查询第一页  每页10个
var result = sqlite.table('user').pages(1);//查询第一页  每页10个默认也是10

result.data;//查询到的数据
result.field;//字段
result.get(0,'name');//获取索引0的name字段数据 方便开发者快速获取
result.pages;//分页是其它数据包含如下
{max:4,current:3,limit:3}
max//最大页数
current//当前页数
limit//每页显示数量
```



## 原始的查询和执行

```javascript
//如果还是没有您需要的你可以自己编写sql使用以下的方法执行

sqlite.query(sql);//执行并返回
sqlite.query(sql,bind);//执行并返回  (它有一个可选参数绑定就是预处理方式)

result.data;//查询到的数据
result.field;//字段
result.get(0,'name');//获取索引0的name字段数据 方便开发者快速获取


//还有执行不返回的
sqlite.execute(sql);
sqlite.execute(sql,bind);//使用方式和上面雷同
```

