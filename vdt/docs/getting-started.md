`Vdt`：一个基于虚拟DOM的模板引擎

## 功能特性 

* 基于虚拟DOM，更新速度快
* 能实现前后端模板继承，包含，宏定义等
* 文件大小在gzip压缩后大概13KB（包含浏览器实时编译模块）
* 支持前后端渲染


<!-- -->
* <!-- {.example-template} -->
    ```jsx 
    // @file ./layout.vdt
    <div>
        <header>
            <b:header />
        </header>
        <div>
            <b:body>
                <div>parent content</div>
            </b:body>
        </div>
    </div>
    ```

    ```jsx
    var layout = require('./layout.vdt');

    <t:layout>
        <b:header>
            <h1>{title}</h1>
        </b:header>
        <b:body>
            <div ev-click={onClick.bind(this)}>Clicked: {count}</div>
            {parent()}
            <ul v-for={items}>
                <li>{key}: {value}</li>
            </ul>
        </b:body>
    </t:layout>
    ```
* <!-- {.example-js} -->
    ```js
    var vdt = Vdt(template);
    vdt.render({
        title: 'vdt',
        items: {
            a: 1,
            b: 2
        },
        count: 0,

        onClick: function() {
            this.data.count++;
            this.update();
        }
    });
    ```
<!-- {ul:.example.dom} -->

## 安装

### 通过script标签引入

Vdt会暴露全局变量`Vdt`，请到[github](https://github.com/Javey/vdt.js/tree/master/dist)下载对应的文件，
或者通过bower安装，然后script标签引入

```bash
bower install vdt --save
```

```html
<script type="text/javascript" src="path/to/vdt.js"></script>
```

### 与webpack & browserify结合使用

使用npm方式安装依赖

```bash
npm install vdt --save
```

```js
var Vdt = require('vdt');
```

### 与requireJs等模块加载器结合使用

Vdt打包的文件支持通过UMD方式加载

```js
define(['path/to/vdt'], function(Vdt) { });
```

## 使用

### 浏览器

```js
var vdt = Vdt('<div>{name}</div>');

// 第一次渲染 
vdt.render({name: 'Vdt'});

// 更新 
vdt.update({name: 'Javascript'});
// 或者，这样修改数据后调用更新
vdt.data.name = 'Javascript'
vdt.update();
```

### NodeJs

* 作为[Express][2]的middleware，用于实时编译Vdt模板

    当Vdt和RequireJs等前端模块加载器结合使用，通常需要NodeJs实时编译Vdt模板，然后当做AMD模块返回

    Vdt提供的middleware`Vdt.middleware`会根据当前请求的js文件路径查找相应目录下是否存在`*.js`文件，如果存在则不处理；
    不存在时会判断该目录下是否存在`*.vdt`文件，如果存在则编译后当做js返回
    ```js
    var Vdt = require('vdt');
    app.use(Vdt.middleware({
        // Vdt模板路径，默认：process.cwd()
        src: 'vdt/template/path', 
        // 是否包成AMD模块，默认：是
        amd: true, 
        // 是否每次都强制编译，否则只有模板文件变更才编译，默认：否
        force: false, 
        // 模板语法分隔符，默认：一对大括号{} 
        delimiters: ['{', '}'], 
        // 自定义文本过滤器，可以对编译结果进行相应处理
        filterSource: function(source) {
            return source
        }
    }));
    ```

* 作为NodeJs的模板引擎

    直接渲染字符串

    ```js
    Vdt('<div>{title}</div>').renderString({title: 'Vdt'});
    // <div>Vdt</div>
    ```
    
    在server中，一般都是渲染文件，可以使用如下方式渲染。首先通过`Vdt.setDefaults()`方法
    设置你的模板路径`views`(默认：views)，以及模板文件扩展名`extname`(默认：vdt)

    ```js
    Vdt.setDefaults({views: 'views', extname: 'vdt'});
    Vdt.renderFile('index', {title: 'Vdt'});
    // <!DOCTYPE html>
    // <div>Vdt</div>
    ```

    `renderFile`方法会自动在添加`doctype`申明，所以你的模板文件中不需要写该申明，否则编译报错。
    如果你想去掉该申明，可以如下设置：

    ```js
    Vdt.setDefaults({doctype: ''});
    ```

* 与[Express](2)结合

    Vdt提供`__express`方法使之可以很方便地作为Express的模板引擎使用

    ```js
    var Express = require('express'),
        Vdt = require('vdt');

    var app = Express();
    app.engine('vdt', Vdt.__express);
    app.set('views', 'views');
    app.set('view engine', 'vdt');

    app.get('/', function(req, res, next) {
        res.render('index', {title: 'Vdt'});
    });
    ```


[1]: https://github.com/Matt-Esch/virtual-dom
[2]: http://www.expressjs.com.cn/
