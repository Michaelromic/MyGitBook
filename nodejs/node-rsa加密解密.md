# [node-rsa加密解密用法](https://tomoya92.github.io/2018/10/12/node-rsa/)

## 生成公钥私钥

在 `node-rsa-demo` 下新建一个文件 `index.js` 写上如下代码
```
var NodeRSA = require('node-rsa')
var fs = require('fs')

function generator() {
  var key = new NodeRSA({ b: 512 })
  key.setOptions({ encryptionScheme: 'pkcs1' })

  var privatePem = key.exportKey('pkcs1-private-pem')
  var publicPem = key.exportKey('pkcs1-public-pem')

  fs.writeFile('./pem/public.pem', publicPem, (err) => {
    if (err) throw err
    console.log('公钥已保存！')
  })
  fs.writeFile('./pem/private.pem', privatePem, (err) => {
    if (err) throw err
    console.log('私钥已保存！')
  })
}

