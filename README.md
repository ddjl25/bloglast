新生项目课程：云计算环境下的博客系统开发
## 实验概述

1. 实验名称：简单博客系统的制作
  
2. 实验目的：学习并掌握HTML、CSS和JavaScript的基础知识，能够独立制作一个简单的博客系统。
  
3. 实验环境：操作系统：Linux
  

                     编程工具：文本编辑器/Visual Studio

                     浏览器：Edge浏览器

## 实验内容

1. 使用HTML构建博客系统的基本结构；
  
2. 使用CSS对博客系统进行样式设计；
  
3. 使用JavaScript实现博客系统的交互功能。
  

## 实验步骤

1.       通过JavaScript，实现博客系统最基本的功能，包括增，删，改，查。

2.       引入css库，并添加各个文段属性使blog更加美观。

3.       给博客系统的内容页添加更多的功能，包括显示日期并将blog按保存时间排序。

4.       给博客系统增加markdown和read more，并渲染markdown，调用marked库

##### 数据库结构
```mermaid
graph LR
  blog[blog]-->ar[articles]-..->id
  ar-..->title
  ar-..->description
  ar-..->markdown
  ar-..->createdAt
  ar-..->HTML
  ```


##### 代码具体讲解
###### 基础功能
以增加为例
```js
<html lang="en">

<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="stylesheet" type="text/css" href="/css/bootstrap.min.css" />
<meta http-equiv="X-UA-Compatible" content="ie=edge">
<title>New Article</title>
</head>

<body>
<div class="container">

<h1 class="mb-4">New Article</h1>

<form action="/new" method="POST">
    <div class="form-group">
    <label for="title">Title</label>
    <input required type="text" name="title" id="title" class="form-control">
    </div>

    <div class="form-group">
    <label for="description">Description</label>
    <textarea name="description" id="description" class="form-control"></textarea>
    </div>

    <div class="form-group">
    <label for="markdown">Markdown</label>
    <textarea required name="markdown" id="markdown" class="form-control"></textarea>
    </div>

    <a href="" class="btn btn-secondary">Cancel</a>
    <button type="submit" class="btn btn-primary">Save</button>
</form>
</div>
</body>
</html>
```
通过表单的形式，以POST方法来进行文章的新增，通过label来标识输入区域的性质，在标题区域用input实现单行输入，而在内容部分用textarea进行多文本输入，同时增加多个css属性，贴近一般博客系统的输入模式
```js
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Blog</title>
  <link rel="stylesheet" type="text/css" href="/css/bootstrap.min.css" />
</head>
<body>

<div class="container">
    <h1  class="mb-4">My blog</h1>
        <a href="/new" class="btn btn-success">New Article</a>
    
    <% articles.forEach(article => { %>
       <div class="card mt-4">
        <div class="card-body">

        <h3 class="card-title"><%= article.title %></h3>

        <div class="card-subtitle text-muted mb-2">
            <%= article.createdAt.toLocaleDateString() %>
        </div>

        <div class="card-text mb-2"><%= article.description %></div>

        <a href="/display/<%= article._id %>" class="btn btn-primary">Read More</a>
        
        <a href="/edit/<%= article._id %>" class="btn btn-info">Edit</a>

        <form action="/<%= article._id %>?_method=DELETE" method="POST" class="d-inline"  >
            <button type="submit" class="btn btn-danger">Delete</button>
        </form>

        </div>
      </div>
    <% }) %>
</div>
</body>
</html>
```
在博客主页我将所有的文章的title和description渲染到这个页面，可以点击read more按钮查看具体内容，此外还有编辑和删除按钮，删除时可以返回post请求。

###### css增加
我在views目录下的display，all，new文件连接了具体的css库
```js
    <link rel="stylesheet" type="text/css" href="/css/bootstrap.min.css" />
   
```
在这些CSS样式中，我在给文章标题内容改变格式，设置了页边距可读性更强，同时我也为各个按钮和输入框设置了独特的样式，使得其看起来简洁而美观。

###### read more功能实现
```js
all.ejs
<a href="/display/<%= article._id %>" class="btn btn-primary">Read More</a>//用户点击这个链接后，将会被导航到服务器上的/display/article._id路径，这通常是由app.get('/display/:id', ...)路由处理函数来响应

server.js
app.get('/display/:id', async (req, res) => {
    const one = await article.findOne({ _id: req.params.id });//响应客户端的GET请求，根据请求ID查找对应的文章，然后渲染一个页面来展示这篇文章的详细信息
    res.render('display',{ article: one });
})
```

###### 日期增加
```js
all.ejs
   <div class="card-subtitle text-muted mb-2">
            <%= article.createdAt.toLocaleDateString() %>
   </div>

server.js
   createdAt: {type: Date,default: Date.now},
```
在server内新建属性createdAt，在all页面插入了article.createdAt的值，通过toLocaleDateString()函数，将日期对象转换为本地日期格式的字符串。

###### 日期排序
```js
app.get('/', async (req, res) => {
  const all = await article.find().sort({ createdAt: 'desc' });
  res.render('all', { articles: all })
})
```
接受到get请求后，在这里使用sort函数，将all页面所有文章按照时间排序

###### 增加markdown属性
```js
server.js
markdown:String,//创建markdown属性

app.post('/new', async (req,res) => {
    one = new article({ title: req.body.title, description: req.body.description,markdown: req.body.markdown });
    await one.save();
    res.render('display', { article: one })
})//创建一个新的文章对象，保存到数据库，并渲染一个页面来显示这个新文章

new.ejs   edit.ejs
<div class="form-group">
    <label for="markdown">Markdown</label>
    <textarea required name="markdown" id="markdown" class="form-control"></textarea>
    </div>//添加markdown属性
```
###### 实现markdown功能
```js
server.js
   const marked = require('marked');//定义marked函数库

   html:String//创建html属性

   articleSchema.pre('validate', function(next) {
  if (this.markdown) {
    this.html = marked(this.markdown)
  }
 next()
})//在数据保存到数据库之前，检查markdown文本，调用函数自动转换为html并保存

app.put('/:id', async (req, res) => {
    let data = {}
    data.title = req.body.title
    data.description = req.body.description
    data.markdown = req.body.markdown

    var one = await article.findOne({ _id: req.params.id });
    if (one != null) {
        one.title = data.title;
        one.description = data.description;
        one.markdown = data.markdown;
        await one.save();       
    }  
    // res.render('display', { article: data })
    res.redirect(`/display/${req.params.id}`);//防止重复的get请求，使用新功能derict，在更新操作完成后，重定向到/display/:id路径
})//用户通过put请求提交的文章更新信息，查找对应的文章对象，更新其内容，并保存回数据库

display.ejs

<p><%- article.html %></p>//讲markdown属性改为html属性，实现功能
  ```

## 实验结果

通过以上步骤，我们制作出了一个简单而美观，基础功能齐全博客系统。

## 实验总结

1. 学会了基本的HTML，CSS，JavaScript语法
  
2. 学会了linux系统的一些基本指令
  
3. 学会了markdown语言的基本语法
  
4. 学会了如何通过HTML建构网页，如何使用CSS美化网页，如何通过JavaScript实现网页交互。
  
5. 学会了数据库的一些基本操作
  
6. 学会了如何在浏览器端调试自己的前端和后端代码，学会了如何通过网络状态和控制台信息来解决程序报错问题
  

## 实验心得

1.学习过程：我通过由简到繁的设计和由易到难的编程实践，逐步掌握了网页开发的基础知识，包括HTML、CSS和JavaScript的基本语法。

2.技能提升：我的编程能力在不断发现问题和解决问题的过程中得到了显著的提升。

3.知识积累：我在网页设计方面实现了从无到有的转变，这表明我积累了一定的知识和技能。

4.热情增长：随着博客系统功能的完善和页面美化，我对编程的热情也在增长。

5.实践成果：通过实验，我不仅提升了技术能力，还增强了对编程的兴趣和热情，这对我的个人成长和职业发展都是有益的。

## 参考文献

1. 教程：[w3school 在线教程](https://www.w3school.com.cn/)
  
2. CSS样式参考：bootstrap.min.css
  

| 工作量统计表 | 基础功能 | 新增功能1 | 新增功能2 | 新增功能3 | 新增功能4 | 新增功能5 | 新增功能6 | 新增功能7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 描述  | 对博客系统中博文的增删改查操作 | 原博客系统CSS美化 | 增加时间属性 | 增加markdown输入框 | 增加read more功能 | 实现markdown功能 | 在display页面渲染markdown | 文章日期排序功能 | 
| 学时  | 8   | 2   | 2   | 2   | 3   | 3   | 2   | 1   |
