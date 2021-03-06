本文主要是使用ReactJs和Redux来实现一个聊天功能的页面，页面极其简单。使用React时间不长，还是个noob，有不对之处欢迎大家吐槽指正。

还要指出这里没有使用到websocket等技术来实现后端逻辑，而是使用了wilddog充当后端。具体关于wilddog的介绍，[戳这里](https://www.wilddog.com/)。

**目标：我期望的页面长这样，一个简单的消息列表，下面有个输入框和提交按钮，任何人可以在上面说话，并可以看到别人说的话，就这么简单。**

![](http://sugarball.me/content/images/2016/03/D0C10F48-8677-4281-94C5-FE7BFE993452.png)
**1. React和Redux**

React这么火，我就不多说了。Redux是一个类flux的应用框架，和flux一样是单向数据流，目前用来主要是对flux进行了一些优化，如将action变为简单的对象，store也是一个简单的对象树，引入了reducer来处理action，reducer本身是pure function，调试起来也极为方便，还可以配合使用hot-loader以及redux-dev-tool实现time travel调试功能，用起来还是有点爽爽的感觉。

**2. 开始coding**

"不想看扯淡，只要看源码的"[戳这里](https://github.com/Sugarball/sugarball.github.io/tree/master/sugarball-chat):D。

文件目录是这样的：

![alt](http://sugarball.me/content/images/2016/03/470D2793-39B9-4E79-922E-8DDA71806575.png)

项目是使用webpack打包的，也用了dev-server，有兴趣的同学可以自己看webpack目录和server.js。

这里主要关注src下面的内容：

1. store：创建一个唯一的store，存放项目中所有的数据。
2. reducers: 初始化store的部分内容，在这里是个空对象。还有处理store的函数，这里只有一个init的工作，就是用action中传过来的chats字段替换当前的state，这里的业务逻辑对应的是，每当聊天室有新的消息传过来，都会整个替换当前的聊天内容，这里可能会有疑问为什么要这样，主要是因为wilddog传给我的就是一个完整的列表，后面仔细会介绍wilddog。
 ![alt](http://sugarball.me/content/images/2016/03/A0824036-A58B-4F5B-A7FF-9CA2B54AF04A.png)
3. action : 只是一个简单的表示操作动作的对象。
 ![alt](http://sugarball.me/content/images/2016/03/841086AF-B725-4DEF-A489-292FD603BE8F.png)
很显然，除了init是简单的对象之外，还有两个相对复杂的函数。这里用到redux-thunk，其作用就是把一些比较复杂的动作(例如这里用到的异步操作)封装到一个action中去，redux-thunk是redux的一个middleware，redux的dispatch无法处理对象之外的内容，例如函数，将redux-thunk放进去了就可以了：

![alt](http://sugarball.me/content/images/2016/03/0511D19A-333E-4976-879A-0A5917A6FEA4.png)

 这里的getChats()首先新建了一个wilddog实例，连接我在wilddog上定义好的项目上，
 ![alt](http://sugarball.me/content/images/2016/03/27E1ACBF-FB43-4BFF-BB7B-964B7B8AC76E.png)

 野狗提供的on()函数可以通过websocket来监听数据是否发生了变化，正如这里写的，每当数据发生变化都会dispatch一次init动作，来将页面的数据重新渲染。

 而postMsg则是将新的消息推到wilddog上。这样会出发wilddog的数据变化，然后反过来会触发我们之前定义在on()回调里面的内容，及重新获取数据，渲染页面。


4.container/chat.js: 页面主体的react组件。首先将这个react组件和redux的store关联起来，这里用到的react-redux中的connect函数，在注解里完成了：
 ![alt](http://sugarball.me/content/images/2016/03/A1441E06-C263-47B6-BB8E-B41409F0A0D1.png)

 大家是否还记得我们之前定义好了获取页面初始消息列表的getChats函数，在这里被调用：
 ![alt](http://sugarball.me/content/images/2016/03/EBE97EEA-B1B4-4B12-825E-A868B459D9AA.png)

 让我们来关注页面html的代码，首先是消息列表：
 ![alt](http://sugarball.me/content/images/2016/03/187C076D-2A87-4C03-AAE0-C728C409A903.png)

 循环_msgList来输出每一条消息，这里用到List,ListItem是material-ui中的。这个_msgList是从store中取出的：

 ![alt](http://sugarball.me/content/images/2016/03/020B8881-C538-4FB4-B9F3-B02570F1000C.png)

 接下来是输入框和say按钮部分的代码：

 ![alt](http://sugarball.me/content/images/2016/03/A17E1DD5-63BC-4870-B94B-057FFBCDD99D.png)

 可以看到是主要是一个表单提交的工作，handleSubmit即表单提交\
 的函数：

 ![alt](http://sugarball.me/content/images/2016/03/6E9E9BDA-7EB1-4A76-855F-0D5E2DC5A217.png)
这里最重要的是this.props.postMsg，调用的是之前定义在action中的postMsg函数完成新增消息的操做。

到目前位置代码部分就完了，这里可以试试[demo](http://sugarball.github.io/sugarball-chat/#/chat)，我用的wilddog个人免费版，资源有限，别整挂了:P






