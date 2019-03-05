# Node连接MySQL并封装其增删查改

## 调用方法
1. 连接Mysql



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


