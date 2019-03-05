# [Nodejs连接mysql的增、删、改、查操作](https://www.cnblogs.com/niejunchan/p/6439924.html)

### 一、准备

nodejs的教程，大多以操作mongodb为示例。但是mongodb有一些局限性，具体官网上有说。我打算用MySQL，因为多少还有点使用经验。先以研究为主。node-mysql，是目前最火的node下的mysql驱动。初步了用了一下，因为异步回调的这种方式，果然好多坑。

下面这个项目的package name是 mysql，版本是mysql@ 2.5.4

先说明下面的所示代码，均已以下代码开头，后面不在说明

var connection = mysql.createConnection({

  host     : '127.0.0.1',

  user     : 'root',

  password : 'root123',

  port: '3306',

  database: 'my_news_test',

});

代码什么意思很直白，如果想深入，可以去上面的官网查。像host，user之类的配置，写过MySQL数据库应用程序的，应该都很清楚，请自行修改相应参数。后面的代码，假定数据库”my_news_test”中有一个叫node_use的表，表有3个属性

id: 自增主键

name：名字，有unique的限制

age

测试MySQL　　MySQL版本：5.5

二、建库并插入5条记录

Source Database       : my_news_test

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------

-- Table structure for node_user

-- ----------------------------

DROP TABLE IF EXISTS `node_user`;

CREATE TABLE `node_user` (

  `id` int(11) NOT NULL AUTO_INCREMENT,

  `name` varchar(30) DEFAULT NULL,

  `age` int(8) DEFAULT NULL,

  PRIMARY KEY (`id`)

) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;

-- ----------------------------

-- Records of node_user

-- ----------------------------

INSERT INTO `node_user` VALUES ('1', 'admin', '32');

INSERT INTO `node_user` VALUES ('2', 'dans88', '45');

INSERT INTO `node_user` VALUES ('3', '张三', '35');

INSERT INTO `node_user` VALUES ('4', 'ABCDEF', '88');

INSERT INTO `node_user` VALUES ('5', '李小二', '65');

三、先测试一下环境

1、  首先需要安装nodejs 的mysql包

D:\User\myappejs4>npm install mysql

mysql@2.5.4 node_modules\mysql

├── require-all@0.0.8

├── bignumber.js@1.4.1

└── readable-stream@1.1.13 (inherits@2.0.1, string_decoder@0.10.31, isarray@0

.0.1, core-util-is@1.0.1)

2、编写nodejs与mysql交互的代码

//mysql.js

//首先需要安装nodejs 的mysql包

//npm install mysql

//编写nodejs与mysql交互的代码

var mysql = require('mysql'); 

var TEST_DATABASE = 'my_news_test'; 

var TEST_TABLE = 'node_user'; 

//创建连接 

var client = mysql.createConnection({ 

  user: 'root', 

  password: 'root123', 

}); 

client.connect();

client.query("use " + TEST_DATABASE);

client.query( 

  'SELECT * FROM '+TEST_TABLE, 

  function selectCb(err, results, fields) { 

    if (err) { 

      throw err; 

    } 

       if(results)

      {

          for(var i = 0; i < results.length; i++)

          {

              console.log("%d\t%s\t%s", results[i].id, results[i].name, results[i].age);

          }

      }   

    client.end(); 

  } 

);

3、运行结果

D:\User\myappejs4>node mysqltest.js

1       admin   32

2       dans88  45

3       张三    35

4       ABCDEF    88

5       李小二  65

[![Nodejs连接mysql的增、删、改、查操作](http://s1.sinaimg.cn/mw690/001EG6RRgy6OVismU1y50&690 "Nodejs连接mysql的增、删、改、查操作")](http://photo.blog.sina.com.cn/showpic.html#blogid=5a6efa330102vctw&url=http://album.sina.com.cn/pic/001EG6RRgy6OVismU1y50)

四、Node.js结合MySQL的增、删、改、查操作

1、增

var mysql  = require('mysql'); 

var connection = mysql.createConnection({    

  host     : '127.0.0.1',      

  user     : 'root',             

  password : 'root123',      

  port: '3306',                  

  database: 'my_news_test',

});

connection.connect();

var  userAddSql = 'INSERT INTO node_user(id,name,age) VALUES(0,?,?)';

var  userAddSql_Params = ['Wilson', 55];

//增 add

connection.query(userAddSql,userAddSql_Params,function (err, result) {

        if(err){

         console.log('[INSERT ERROR] - ',err.message);

         return;

        }       

       console.log('-------INSERT----------');

       //console.log('INSERT ID:',result.insertId);       

       console.log('INSERT ID:',result);       

       console.log('#######################'); 

});

connection.end();

运行如下

D:\User\myappejs4>node mysqltestadd.js

-------INSERT----------

INSERT ID: { fieldCount: 0,

  affectedRows: 1,

  insertId: 6,

  serverStatus: 2,

  warningCount: 0,

  message: '',

  protocol41: true,

  changedRows: 0 }

#######################

 [![Nodejs连接mysql的增、删、改、查操作](http://s7.sinaimg.cn/mw690/001EG6RRgy6OViQwQAed6&690 "Nodejs连接mysql的增、删、改、查操作")](http://photo.blog.sina.com.cn/showpic.html#blogid=5a6efa330102vctw&url=http://album.sina.com.cn/pic/001EG6RRgy6OViQwQAed6)

2、改

var mysql  = require('mysql'); 

var connection = mysql.createConnection({    

  host     : '127.0.0.1',      

  user     : 'root',             

  password : 'root123',      

  port: '3306',                  

  database: 'my_news_test',

});

connection.connect();

var userModSql = 'UPDATE node_user SET name = ?,age = ? WHERE id = ?';

var userModSql_Params = ['Hello World',99,7];

//改 up

connection.query(userModSql,userModSql_Params,function (err, result) {

   if(err){

         console.log('[UPDATE ERROR] - ',err.message);

         return;

   }       

  console.log('----------UPDATE-------------');

  console.log('UPDATE affectedRows',result.affectedRows);

  console.log('******************************');

});

connection.end();

运行结果如下

D:\User\myappejs4>node mysqltest_up.js

----------UPDATE-------------

UPDATE affectedRows 1

******************************

3、查操作

var mysql  = require('mysql'); 

var connection = mysql.createConnection({    

  host     : '127.0.0.1',      

  user     : 'root',             

  password : 'root123',      

  port: '3306',                  

  database: 'my_news_test',

});

connection.connect();

var  userGetSql = 'SELECT * FROM node_user';

//查 query

connection.query(userGetSql,function (err, result) {

        if(err){

          console.log('[SELECT ERROR] - ',err.message);

          return;

        }       

       console.log('---------------SELECT----------------');

       console.log(result);       

       console.log('$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$'); 

});

connection.end();

运行的结果如下

D:\User\myappejs4>node mysqltest_query.js

---------------SELECT----------------

[ { id: 1, name: 'admin', age: 32 },

  { id: 2, name: 'dans88', age: 45 },

  { id: 3, name: '张三', age: 35 },

  { id: 4, name: 'ABCDEF', age: 88 },

  { id: 5, name: '李小二', age: 65 },

  { id: 6, name: 'Wilson', age: 55 } ]

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

 [![Nodejs连接mysql的增、删、改、查操作](http://s3.sinaimg.cn/mw690/001EG6RRgy6OViDwYHU02&690 "Nodejs连接mysql的增、删、改、查操作")](http://photo.blog.sina.com.cn/showpic.html#blogid=5a6efa330102vctw&url=http://album.sina.com.cn/pic/001EG6RRgy6OViDwYHU02)

4、删除操作

var mysql  = require('mysql'); 

var connection = mysql.createConnection({    

  host     : '127.0.0.1',      

  user     : 'root',             

  password : 'root123',      

  port: '3306',                  

  database: 'my_news_test',

});

connection.connect();

var  userDelSql = 'DELETE FROM node_user WHERE id = 7';

//ɾ

connection.query(userDelSql,function (err, result) {

        if(err){

          console.log('[DELETE ERROR] - ',err.message);

          return;

        }       

       console.log('-------------DELETE--------------');

       console.log('DELETE affectedRows',result.affectedRows);

       console.log('&&&&&&&&&&&&&&&&&'); 

});

connection.end();

运行的结果如下

D:\User\myappejs4>node mysqltest_del.js

-------------DELETE--------------