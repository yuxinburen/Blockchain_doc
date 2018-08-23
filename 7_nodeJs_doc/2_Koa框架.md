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

#### 三，中间件

Koa最大的特色，也是最重要的一个设计，就是中间件(Middleware)。最简单的写法为在`main`函数里面增加一行。

`console.log(${Date.now()}${ctx.request.method}${ctx.request.url})`

##### 3.2中间件概念

```
const logger = (ctx,next) =>{
    console.log(${Date.now()}${ctx.request.method}${ctx.request.url});
    next();
}
app.use(logger);
```

类似如上代码块的函数就是“中间件”，因为他处在Http request 和Http Response中间，用来实现某种中间功能。app.use()用来加载中间件。

基本上，Koa所有的功能都是通过中间件实现的。每个中间件默认接受两个参数，第一个参数是Context对象，第二个参数是next函数。只要调用next函数，就可以把执行权交给下一个中间件。

##### 3.3中间件栈

多个中间件会形成一个栈结构，以“先进后出”的顺序执行。

1. 最外层的中间件首先执行。
2. 调用next函数，把执行权交给下一个中间件。
3. ...
4. 最内层的中间件最后执行。
5. 执行结束后，把执行权交回上一层的中间件。
6. ...
7. 最外层的中间件收回执行权之后，执行next函数后面的代码。

##### 3.4异步中间件

在中间件中，如果有异步操作（比如读取数据库），中间件就必须写成async函数。

```
const main ＝ async function(ctx, next){
    ctx.response.type = "html";
    ctx.response.body = await.fs.readFile("./demos/template.html","utf8");
};
app.use(main);
```

以上代码中，fs.readFile是一个异步操作，必须写成await fs.readFile(),然后中间件必须写成async函数。

##### 3.5中间件的合成

`koa-compose`模块可以将多个中间件合成为一个。如下代码：

```
const compose = require("koa-compose");
const logger = (ctx,next) =>{
    console.log(`${Date.now()}${ctx.request.method}${ctx.request.url}`);
    next();
}
const main = ctx =>{
    ctx.response.body = "hello world";
};
const middleware = compose([logger,main]);
app.use(middleware);
```

#### 四，错误处理

如果代码执行过程中出现错误，我们需要把错误信息返回给用户。Http协议约定时要返回500状态码。Koa提供了ctx.throw()方法，用来抛出错误，ctx.throw(500)就是抛出500错误。如下示例：

```
const main ＝ ctx =>{
    ctx.throw(500);
}
```

##### 4.2 404错误

如果将`ctx.response.status`设置成404，就相当于`ctx.throw(404)`，返回404错误。

```
const main = ctx =>{
    ctx.response.status = 404;
    ctx.response.body = "Page Not Found";
};
```

##### 4.3处理错误的中间件

为了方便处理错误，最好使用`try...catch`进行捕获。但是，为每个中间件都写`try...catch`比较麻烦，我们可以让最外层的中间件，负责所有的中间件的错误处理。

```
const handler ＝ async (ctx,next) =>{
    try{
        await next();
    }catch(err){
        ctx.response.status = err.statusCode || err.status || 500;
        ctx.response.body = {
            message:err.message
        };
    }
};

const main = ctx =>{
    ctx.throw(500);
};

app.use(handler);
app.use(main);

```

##### 4.4 error 事件的监听

运行过程中一旦出错，Koa会触发一个error事件。监听这个事件，也可以处理错误。示例如下：

```
const main = ctx =>{
    ctx.throw(500);
};
app.on("error",(err,ctx)=>{
    console.error("server error",err);
});
```

##### 4.5释放error事件

需要注意的是，如果错误被`try...catch`捕获，就不会触发error事件。此时，必须调用ctx.app.emit();手动释放error事件，才能让监听函数生效。

```
const handler = async (ctx,next) =>{
    try{
        await next();
    }catch(err){
        ctx.response.status = err.statusCode || err.status || 500;
        ctx.response.type = 'html';
        ctx.response.body = 'Something wrong, please contract  administrator.';
        ctx.app.emit("error",err,ctx);
    }
};
const main = ctx =>{
    ctx.throw(500);
};
app.on('error',function(err){
    console.log('logging error',err.message);
    console.log(err);
})
```

##### 5.1Cookies功能

ctx.cookies可以设置读写的Cookie。

```
const main = function(ctx){
    const n = Number(ctx.cookies.get("view")||0)+1;
    ctx.cookies.set("view",n);
    ctx.response.body = n+"views";
}
```

Cookie功能就是两个方法的使用，ctx.cookies.set()方法以及ctx.cookies.get();

##### 5.2表单

Web应用离不开处理表单。本质上，表单的数据提交就是通过POST方法发送到服务器的键值对。koa-body模块可以用来从POST请求的数据体里面提取键值对。

```
const koaBody ＝ require("koa-body");
const main = async function(ctx){
    const body = ctx.request.body;
    if(!body.name) ctx.throw(400,".name required");
    ctx.body ={name:body.name};
};

app.use(koaBody());
```

##### 2.3文件上传

koa-body模块还可以用来处理文件上传。代码示例如下所示：

```
const os = require("os");
const path = require("path");
const koaBody = require("koa-body");

const main = async function(ctx) {
    const tmpdir = os.tmpdir();
    const filePaths = [];
    const files = ctx.request.body.files || {};
    
    for(let key in files){
        const file = files[key];
        const filePath = path.join(tmpdir,file.name);
        const reader = fs.createReader(file.path);
        const writer = fs.createWriteStream(filepath);
        reader.pipe(writer);
        filePaths.push(filepath);
    }
    ctx.body = filepaths;
};
app.use(koabody({multipart:true}));
```

