#### Koa库的说明及使用

##### Koa介绍

Node主要用来开发web应用。因此，使用Node，往往需要一些web应用框架。

Koa是一种简单好用的Web框架。Koa的特点是优雅，简洁，表达力强，自由度高。

##### Context对象

Koa提供一个Context对象，表示一次Http请求对话的上下文，通过加工这个对象，就可以控制返回给用户的内容。

Context.response.body属性就是发送给用户的内容，如下示例：

```
const Koa = require("koa")
const app = new Koa()

const main = ctx =>{
    ctx.response.body = 'hello world';
}
app.use(main)
app.listen(3000)
```

解释：如上的代码，main函数用来设置ctx.response.body。然后，使用app.use方法加载main函数。ctx.response代表HTTP Response。同样的，ctx.request代表HTTP Request。

##### HTTP Response的类型

Koa默认的返回类型是`text/plain`，如果想返回其他类型的内容，可以先用ctx.request.accepts判断一下，客户端希望接受什么数据（根据HTTP Request的Accept字段），然后使用ctx.response.type指定返回类型。

##### 网页模版

实际开发中，返回给用户的网页往往都写成模版文件。我们可以让Koa先读取模版文件，然后将这个模版返回给用户。例如：

```
const fs = require("fs")
const main = ctx => {
    ctx.response.type = "html";
    ctx.response.body = fs.createReadStream("./demos/templete.html");
};
```

#### 二，路由

##### 2.1原生路由

网站一般由多个页面。通过ctx.request.path可以获取到用户请求的路径，由此实现简单的路由，如下所示：

```
const main = ctx =>{
    if(ctx.request.path != '/') {
        ctx.response.type = "html";
        ctx.response.body = "<a href='/'> index page </a>";
    } else {
        ctx.response.body = "hello world";
    }
};
```

##### 2.2koa-route模块

原生路由使用比较麻烦，可以使用封装好的route模块，如下所示：

```
const route = require("koa-route");

const about = ctx =>{
    ctx.response.type = "html";
    ctx.response.body = "<a href='/'>index page </a>";
};

const main = ctx =>{
    ctx.response.body = "hello world";
};

app.use(route.get("/",main));
app.use(route.get("/about",about));
```

##### 2.3静态资源

一般的Web网站应用都会用到静态资源（图片，字体，样式表，脚本...)，为他们一个个写路由非常麻烦，也没有必要。在koa框架中，封装了koa-static模块来实现这部分的需求，

##### 2.4重定向

有一些场合，需要进行重定向（redirect）访问请求。比如，登录以后，将他重定向到登录前的页面。ctx.response.redirect()方法可以发出一个302跳转，将用户导向另一个路由。