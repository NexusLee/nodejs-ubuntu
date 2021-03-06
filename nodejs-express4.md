# Nodejs开发框架Express4.X开发

------
##1. 建立工程
  express命令创建工程，支持ejs
    
    ~express -e nodejs-demo
  
  下载依赖包
  
    ~npm install
    
  启动项目
  
    ~npm start
    > nodejs-demo@0.0.0 start /home/nexus/workspace/nodejs-demo
    > node ./bin/www

  本地的3000端口被打开，通过浏览器访问: localhost:3000
  
##2. Express4.X配置文件

  app.js
  
    var express = require('express');
    var path = require('path');
    var favicon = require('static-favicon');
    var logger = require('morgan');
    var cookieParser = require('cookie-parser');
    var bodyParser = require('body-parser');
    
    var routes = require('./routes/index');
    var users = require('./routes/users');
    var http = require('http');
    var ejs =require('ejs');
    
    var app = express();

    // view engine setup
    app.set('views', path.join(__dirname, 'views'));
    app.engine('.html', ejs.__express);
    app.set('view engine', 'html');
    //app.set('view engine', 'ejs');
    
    app.use(favicon());
    app.use(logger('dev'));
    app.use(bodyParser.json());
    app.use(bodyParser.urlencoded());
    app.use(cookieParser());
    app.use(express.static(path.join(__dirname, 'public')));

    app.use('/', routes);
    app.use('/users', users);
    
    
    /// catch 404 and forward to error handler
    app.use(function(req, res, next) {
        var err = new Error('Not Found');
        err.status = 404;
        next(err);
    });

    /// error handlers
    
    // development error handler
    // will print stacktrace
    if (app.get('env') === 'development') {
        app.use(function(err, req, res, next) {
            res.status(err.status || 500);
            res.render('error', {
                message: err.message,
                error: err
            });
        });
    }
    
    // production error handler
    // no stacktraces leaked to user
    app.use(function(err, req, res, next) {
        res.status(err.status || 500);
        res.render('error', {
        message: err.message,
        error: {}
    });
});


module.exports = app;

##3. 增加Bootstrap界面框架

复制到public/stylesheets目录
 
    bootstrap.min.css
    bootstrap-responsive.min.css
    
复制到public/javascripts目录

    bootstrap.min.js
    jquery-1.9.1.min.js
    
接下来，我们把index.html页面切分成3个部分：header.html, index.html, footer.html

header.html, 为html页面的头部区域

index.html, 为内容显示区域

footer.html，为页面底部区域

header.html

    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="utf-8">
    <title><%=: title %></title>
    <!-- Bootstrap -->
    <link href="/stylesheets/bootstrap.min.css" rel="stylesheet" media="screen">
    <!-- <link href="css/bootstrap-responsive.min.css" rel="stylesheet" media="screen"> -->
    </head>
    <body screen_capture_injected="true">
    
index.html

    <% include header.html %>
    <h1><%= title %></h1>
    <p>Welcome to <%= title %></p>
    <% include footer.html %>
    
footer.html

    <script src="/javascripts/jquery-1.9.1.min.js"></script>
    <script src="/javascripts/bootstrap.min.js"></script>
    </body>
    </html>
  
访问localhost:3000。

##4. 路由功能

访问路径：/，页面：index.html，不需要登陆，可以直接访问。

访问路径：/home，页面：home.html，必须用户登陆后，才可以访问。

访问路径：/login，页面：login.html，登陆页面，用户名密码输入正确，自动跳转到home.html

访问路径：/logout，页面：无，退出登陆后，自动回到index.html页面

在app.js文件中增加路由配置

    app.get('/', routes);
    app.get('/login', routes);
    app.post('/login', routes);
    app.get('/logout', routes);
    app.get('/home', routes);

routes/index.js文件，增加对应的方法。

    router.get('/', function(req, res) {
      res.render('index', { title: 'Express' });
    });
    
    router.get('index' , function(req, res){
      res.render('index', { title: 'Express' });
    });
    
    router.get('login', function(req, res){
      res.render('login', { title: '用户登陆'});
    });
    
    function doLogin(req, res){
      var user={
        username:'admin',
        password:'admin'
      }
      if(req.body.username===user.username && req.body.password===user.password){
        res.redirect('/home.html');
      }
      res.redirect('/login.html');
      });

    router.get('logout', function(req, res){
      res.redirect('/');
    });
    
    router.get('home', function(req, res){
      var user={
          username:'admin',
          password:'admin'
      }
    
    res.render('home', { title: 'Home',user: user});
    
    });

创建views/login.html和views/home.html两个文件

login.html

    <% include header.html %>
    <div class="container-fluid">
    <form class="form-horizontal" method="post">
    <fieldset>
    <legend>用户登陆</legend>
    <div class="control-group">
    <label class="control-label" for="username">用户名</label>
    <div class="controls">
    <input type="text" class="input-xlarge" id="username" name="username">
    </div>
    </div>
    <div class="control-group">
    <label class="control-label" for="password">密码</label>
    <div class="controls">
    <input type="password" class="input-xlarge" id="password" name="password">
    </div>
    </div>
    <div class="form-actions">
    <button type="submit" class="btn btn-primary">登陆</button>
    </div>
    </fieldset>
    </form>
    </div>
    <% include footer.html %>

home.html

    <% include header.html %>
    <h1>Welcome <%= user.username %>, 欢迎登陆！！</h1>
    <a claa="btn" href="/logout">退出</a>
    <% include footer.html %>

修改index.html，增加登陆链接

    <% include header.html %>
    <h1>Welcome to <%= title %></h1>
    <p><a href="/login">登陆</a></p>
    <% include footer.html %>

##5. Session使用

使用connect-mongostore配置session

    var express = require('express');
    var session = require('express-session');
    var MongoStore = require('connect-mongostore')(session);
    var app = express();

    app.use(session({
    secret: 'my secret',
    store: new MongoStore({'db': 'sessions'})
    }));

修改routes/index.js文件

    function doLogin(req, res){
      var user={
        username:'admin',
        password:'admin'
      }
      if(req.body.username===user.username && req.body.password===user.password){
        req.session.user=user;
        res.redirect('/home.html');
      }
      res.redirect('/login.html');
    });
    
    function logout(req, res){
      req.session.user=null;
      res.redirect('/');
    };
##6. Mongoose使用

增加mongoose的类库

    cd d:/nexus/node/nodejs-demo
    npm install mongoose

增加models目录，并添加mongodb.js

    var mongoose = require('mongoose');
    mongoose.connect('mongodb://localhost/nodejs');
    exports.mongoose = mongoose;
    
指定Mongo的数据库名为nodejs

models目录，并添加Movie.js
   
    var mongodb = require('./mongodb');
    var Schema = mongodb.mongoose.Schema;
    var MovieSchema = new Schema({
    name : String,
    alias : [String],
    publish : Date,
    create_date : { type: Date, default: Date.now},
    images :{
    coverSmall:String,
    coverBig:String,
    },
    source :[{
    source:String,
    link:String,
    swfLink:String,
    quality:String,
    version:String,
    lang:String,
    subtitle:String,
    create_date : { type: Date, default: Date.now }
    }]
    });
    var Movie = mongodb.mongoose.model("Movie", MovieSchema);
    var MovieDAO = function(){};
    module.exports = new MovieDAO();
    
指定Mongo的数据库集为Movie

app.js增加访问路径

    var movies = require('./routes/movie');
    ...
    app.get('/movie/add',movies);//增加
    app.post('/movie/add',movies);//提交
    app.get('/movie/:name',movies);//编辑查询
    app.get('/movie/json/:name',movies);//JSON数据
    
在routes目录，增加movie.js

    var Movie = require('./../models/Movie.js');
    var express = require('express');
    var router = express.Router();
    router.get('/movie/add' , function(req, res) {
            return res.render('movie',{
                title:'新增加|电影|管理|moive.me',
                label:'新增加电影',
                movie:false,
                content:""
            });
    });
    router.post('/movie/add' , function(req, res) {
        console.log(req.body.content);
        var json = req.body.content;
        if(json._id){//update
        } else {//insert
            Movie.save(json, function(err){
                if(err) {
                    res.send({'success':false,'err':err});
                } else {
                    res.send({'success':true});
                }
            });
        }
    });
    router.get('/movie/:name' , function(req, res) {
        if(req.params.name) {//
           //var obj = movieJSON(req, res);
            //res.send(obj);
            Movie.findByName(req.params.name,function(err, obj){
                return res.render('movie', {
                    title:req.params.name+'|电影|管理|moive.me',
                    label:'编辑电影:'+req.params.name,
                    movie:req.params.name,
                    content:obj
                 });
            });
        }
    });
    module.exports = router;
    
在views目录，增加movie.html

    <% include header.html %>
    <div class="container-fluid">
    <div class="row-fluid">
    <div class="span8">
    <form>
    <fieldset>
    <legend><%=label%></legend>
    <textarea id="c_editor" name="c_editor" class="span12" rows="10"></textarea>
    <button id="c_save" type="button" class="btn btn-primary">保存</button>
    </fieldset>
    <form>
    </div>
    </div>
    </div>
    <% include footer.html %>
