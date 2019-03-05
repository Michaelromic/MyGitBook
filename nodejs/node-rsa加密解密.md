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
```


---
## 上面的例子是私钥加密，公钥解密，下面自测，用公钥加密，私钥解密
```
var cipherText;
function encrypt(data) {
    return new Promise((resolve,reject)=>{
        fs.readFile('./rsa_key/rsa_public_key.pem', function (err, pubkey) {
            if(err){
                reject(console.log(err))
            } else {
                var key = new NodeRSA(pubkey);
                cipherText = key.encrypt(data, 'base64');

                console.log(data);
                console.log(cipherText);
                console.log("111");

                resolve();
            }
        });
    })
}

function decrypt(data) {
    fs.readFile('./rsa_key/rsa_private_key.pem', function (err, privkey) {
        var key = new NodeRSA(privkey);
        let rawText = key.decrypt(data, 'utf8');
        console.log(rawText);
    });
}

async function testrsa()
{
    const data = "cUw4fDSUDXYbFSDp5WUBYhpaKb3cdzaQyBFJVzAH3YJFgNb5dTSM";
    await encrypt(data);
    console.log("222");
    console.log(cipherText);
    decrypt(cipherText);
}

testrsa();
```
---
