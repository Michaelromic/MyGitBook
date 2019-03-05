# [node-rsa加密解密用法](https://tomoya92.github.io/2018/10/12/node-rsa/)

## 生成公钥私钥
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
generator();
```

## 加密

加密 "hello world" 这个字符串
```
function encrypt() {
    fs.readFile('./pem/private.pem', function (err, data) {
        var key = new NodeRSA(data);
        let cipherText = key.encryptPrivate('hello world', 'base64');
        console.log(cipherText);
    });
}

//generator();
encrypt();
```

## 解密

把上一步加密获得的密文复制粘贴到下面要解密的方法内

```
function decrypt() {
fs.readFile('./pem/public.pem', function (err, data) {
var key = new NodeRSA(data);
let rawText = key.decryptPublic('fH1aVCUceJYVvt1tZ7WYc1Dh5dVCd952GY5CX283V/wK2229FLgT9WfRNAPMjbTtwL9ghVeYD4Lsi6yM1t4OqA==', 'utf8');
console.log(rawText);
});
}

//generator();
//encrypt();
decrypt();
```

