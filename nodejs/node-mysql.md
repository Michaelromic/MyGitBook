# [Node连接MySQL并封装其增删查改](https://juejin.im/post/5afd4aeb6fb9a07ab458d78a)

## 调用方法
1. 连接Mysql

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
2. 查询

```
connection.query('SELECT * FROM t_user WHERE username = "whg"', (err, results, fields) => {
    if(err){
        console.log(err);
    }
    console.log(results);
})
```


## 封装
1.数据库配置文件
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


