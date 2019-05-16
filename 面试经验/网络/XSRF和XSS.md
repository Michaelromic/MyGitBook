# XSRF(或叫做CSRF)和XSS
CSRF(Cross-site request forgery)跨站请求伪造：通过伪装来自受信任用户的请求来利用受信任的网站。
XSS(Cross Site Scripting)跨站脚本攻击
CSRF重点在请求,XSS重点在脚本

CSRF：网站过分信任用户，放任来自所谓通过访问控制机制的代表合法用户的请求执行网站的某个特定功能。
XSS：用户过分信任网站，放任来自浏览器地址栏代表的那个网站代码在自己本地任意执行。如果没有浏览器的安全机制限制，xss代码可以在用户浏览器为所欲为
