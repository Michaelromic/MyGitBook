#### 转自： [通俗易懂RESTful，如何设计RESTful风格API](https://blog.csdn.net/a78270528/article/details/78469758)

## 概念
REST -- REpresentational State Transfer 直译：表现层状态转移。这个中文直译经常出现在很多文章中。尼玛，谁听得懂“表现层状态转移”，这是人话吗？

那就逐个单词来理解REST名称
REST -- REpresentational State Transfer
首先，之所以晦涩是因为前面主语被去掉了，全称是 Resource Representational State Transfer：通俗来讲就是：资源在网络中以某种表现形式进行状态转移。分解开来：
Resource：资源，即数据（前面说过网络的核心）。比如 newsfeed，friends等；
Representational：某种表现形式，比如用JSON，XML，JPEG等；
State Transfer：状态变化。通过HTTP动词实现。

## 为什么要用RESTful结构呢？
大家都知道"古代"网页是前端后端融在一起的，比如之前的PHP，JSP等。在之前的桌面时代问题不大，但是近年来移动互联网的发展，各种类型的Client层出不穷，RESTful可以通过一套统一的接口为 Web，iOS和Android提供服务。另外对于广大平台来说，比如Facebook platform，微博开放平台，微信公共平台等，它们不需要有显式的前端，只需要一套提供服务的接口，于是RESTful更是它们最好的选择。

#### 从原理角度来分析：
根据Richardson Maturity Model（理查德森成熟度模型）, REST架构的成熟度有4个等级：
Level 0 - 面向前台
我们在咖啡店向前台点了一杯拿铁，这个过程可以用这段文字来描述：
{
    "addOrder": {
        "orderName": "latte"
    }
}
我们通过这段文字，告诉前台，新增一笔订单，订单是一杯拿铁咖啡，接着，前台给我们返回这么一串回复：
{
    "orderId": "123456"
}
假设我们有一张会员卡，我们想查询一下这张会员卡的余额，这时候，要向前台发起另一个询问：
{
    "queryBalance": {
        "cardId": "447031335"
    }
}
查询卡号为447031335的卡的余额，查询的结果返回来了：
{
    "balance": "0"
}
没钱……
哈哈，没钱，现在我们要跟前台说，这杯咖啡不要了：
{
    "deleteOrder": {
        "orderId": "123456"
    }
}


##### Level 1 - 面向资源
现在这家咖啡店越做越大，来喝咖啡的人越来越多，单靠前台显然是不行的，店主决定进行分工，每个资源都有专人负责，我们可以直接面向资源操作。
比如还是下单，请求的内容不变，但是我们多了一条消息:
/orders
{
    "addOrder": {
        "orderName": "latte"
    }
}
多了一个斜杠和orders，这是什么意思？
这个表示我们这个请求是发给哪个资源的，订单是一种资源，我们可以理解为是咖啡厅专门管理订单的人，他可以帮我们处理所有有关订单的操作，包括新增订单、修改订单、取消订单等操作。
接着还是会返回订单的编号给我们：
{
    "orderId": "123456"
}
下面，我们还是要查询会员卡余额，这次请求的资源变成了cards：
/cards
{
    "queryBalance": {
        "cardId": "447031335"
    }
}
接下来是取消订单：
/orders
{
    "deleteOrder": {
        "orderId": "123456"
    }
}


##### Level2 - 打上标签
接下来，店主还想继续优化他的咖啡厅的服务流程，他发现负责处理订单的员工，每次都要去订单内容里面看是新增订单还是删除订单，还是其他的什么操作，十分不方便，于是规定，所有新增资源的请求，都在请求上面写上大大的‘POST’，表示这是一笔新增资源的请求。
其他种类的请求，比如查询类的，用‘GET’表示，删除类的，用‘DELETE’表示，修改用PATCH表示。
来，我们再来重复上面那个过程，来一杯拿铁：
POST /orders
{
    "orderName": "latte"
}
请求的内容简洁多啦，不用告诉店员是addOrder，看到POST就知道是新增，返回的内容还是一样：
{
    "orderId": "123456"
}
接着是查询会员卡余额，这次也简化了很多：
GET /cards
{
    "cardId": "447031335"
}
这个请求我们还可以进一步优化为这样：
GET /cards/447031335
直接把要查询的卡号写在后面了。
没错，接着，取消订单：
DELETE /orders/123456


##### Level 3 - 完美服务
忽然有一天，有个顾客抱怨说，他买了咖啡后，不知道要怎么取消订单，咖啡厅一个店员回了一句，你不会看我们的宣传单吗，上面不写着：
DELETE /orders/{orderId}
顾客反问道，谁会去看那个啊，店员不服，又说到，你瞎了啊你……后面两人吵着吵着还打了起来… 
噗，真是悲剧…
有了这次教训，店长决定，顾客下了单之后，不仅给他们返回订单的编号，还给顾客返回所有可以对这个订单做的操作，比如告诉用户如何删除订单。现在，我们还是发出请求，请求内容和上一次一样：
POST /orders
{
    "orderName": "latte"
}
但是这次返回时多了些内容：
{
    "orderId": "123456",
    "link": {
        "rel": "cancel",
        "url": "/order/123456"
    }
}
这次返回时多了一项link信息，里面包含了一个rel属性和url属性，rel是relationship的意思，这里的关系是cancel，url则告诉你如何执行这个cancel操作，接着你就可以这样子来取消订单啦：
DELETE /orders/123456
哈哈，这服务真是贴心，以后再也不用担心店员和顾客打起来了。
Level3的Restful API，给使用者带来了很大的遍历，使用者只需要知道如何获取资源的入口，之后的每个URI都可以通过请求获得，无法获得就说明无法执行那个请求。
现在绝大多数的RESTful接口都做到了Level2的层次，做到Level3的比较少。当然，这个模型并不是一种规范，只是用来理解Restful的工具。所以，做到了Level2，也就是面向资源和使用Http动词，就已经很Restful了。


##### Levels的意义
Level 1 解释了如何通过分治法(Divide and Conquer)来处理复杂问题，将一个大型的服务端点(Service Endpoint)分解成多个资源。
Level 2 引入了一套标准的动词，用来以相同的方式应对类似的场景，移除不要的变化。
Level 3 引入了可发现性(Discoverability)，它可以使协议拥有自我描述(Self-documenting)的能力。
这一模型帮助我们思考我们想要提供的HTTP服务是何种类型的，同时也勾勒出人们和它进行交互时的期望。


#### 从应用角度来分析：
一、REST描述的是在网络中client和server的一种交互形式；REST本身不实用，实用的是如何设计 RESTful API（REST风格的网络接口）；
二、Server提供的RESTful API中，URL中只使用名词来指定资源，原则上不使用动词。“资源”是REST架构或者说整个网络处理的核心。
URL定位资源，用HTTP动词（GET,POST,DELETE,DETC）描述操作。
1、看Url就知道要什么
2、看http method就知道干什么
3、看http status  code就知道结果如何
比如：

http://api.qc.com/v1/newsfeed: 获取某人的新鲜; 
http://api.qc.com/v1/friends: 获取某人的好友列表;
http://api.qc.com/v1/profile: 获取某人的详细信息;
三、用HTTP协议里的动词来实现资源的添加，修改，删除等操作。即通过HTTP动词来实现资源的状态扭转：
GET 用来获取资源，
POST 用来新建资源（也可以用于更新资源），
PUT 用来更新资源，
DELETE 用来删除资源。
比如：
DELETE http://api.qc.com/v1/friends: 删除某人的好友 （在http parameter指定好友id）
POST http://api.qc.com/v1/friends: 添加好友
UPDATE http://api.qc.com/v1/profile: 更新个人资料
四、Server和Client之间传递某资源的一个表现形式，比如用JSON，XML传输文本，或者用JPG，WebP传输图片等。当然还可以压缩HTTP传输时的数据（on-wire data compression）。
五、用 HTTP Status Code传递Server的状态信息。比如最常用的 200 表示成功，500 表示Server内部错误等。

## 总结
好了，理解了RESTful的概念，究竟如何应用，这是个问题。根据项目的需求不同，我们的API设计规范也存在差别，完全看自身理解，满足自身需求，大的理念不变，根据需求制定项目的API规范就是好的RESTful。


