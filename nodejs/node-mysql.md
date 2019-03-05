# [Node连接MySQL并封装其增删查改](https://juejin.im/post/5afd4aeb6fb9a07ab458d78a)

## 调用方法
##### 1. 连接Mysql
```
const mysql = require('mysql');

let connection = mysql.createConnection({
    host : 'localhost',
    user : 'root', 
    password : 'password',
    database : 'test'
});

connection.connect(function(err) {
  if (err) {
    console.error('连接失败: ' + err.stack);
    return;
  }
  console.log('连接成功 id ' + connection.threadId);
});
```
##### 2. 查询
```
connection.query('SELECT * FROM t_user WHERE username = "whg"', (err, results, fields) => {
    if(err){
        console.log(err);
    }
    console.log(results);
})
```
##### 3. 添加
```
connection.query('INSERT INTO t_user(username, pass) VALUES(?, ?)',['whg', '123'], (err, results) => {
    if(err){
        console.log(err);
    }
    console.log(results);
})
```
##### 4. 删除
```
connection.query('DELETE FROM t_user  WHERE id = 1', (err, results) => {
    if(err){
        console.log(err);
    }
    console.log(results);
})
```
##### 5. 更新
```
connection.query('UPDATE t_user SET pass = "321" WHERE username = "whg"', (err, results) => {
    if(err){
        console.log(err);
    }
    console.log(results);
})
```
##### 6. 结束连接
```
connection.end(function(err) {
  
});
connection.destroy();

```
---
## 封装
##### 1. 数据库配置文件
```
//配置链接数据库参数
module.exports = {
    host : 'localhost',
    port : 3306,//端口号
    database : 'nodetest',//数据库名
    user : 'root',//数据库用户名
    password : '123456'//数据库密码
};
```
##### 2. 封装、暴露方法
```
let mysql = require('mysql');//引入mysql模块
var databaseConfig = require('./mysql.config');  //引入数据库配置模块中的数据

//向外暴露方法
module.exports = {
    query : function(sql,params,callback){
        //每次使用的时候需要创建链接，数据操作完成之后要关闭连接
        var connection = mysql.createConnection(databaseConfig);        
        connection.connect(function(err){
            if(err){
                console.log('数据库链接失败');
                throw err;
            }
         //开始数据操作
         //传入三个参数，第一个参数sql语句，第二个参数sql语句中需要的数据，第三个参数回调函数
        connection.query( sql, params, function(err,results,fields ){
           if(err){
                console.log('数据操作失败');
                throw err;
            }
            //将查询出来的数据返回给回调函数
            callback && callback(results, fields);
            //results作为数据操作后的结果，fields作为数据库连接的一些字段
            //停止链接数据库，必须再查询语句后，要不然一调用这个方法，就直接停止链接，数据操作就会失败
             connection.end(function(err){
                  if(err){
                      console.log('关闭数据库连接失败！');
                      throw err;
                  }
              });
           });
       });
    }
};

```
##### 3. demo
```
var db=require('../model/mysql.js');
// 查询实例
db.query('select * from t_user', [],function(result,fields){
    console.log('查询结果：');
    console.log(result);
});
//添加实例
var  addSql = 'INSERT INTO websites(username,password) VALUES(?,?)';
var  addSqlParams =['咕噜先森', '666'];
db.query(addSql,addSqlParams,function(result,fields){
    console.log('添加成功')
})

```



