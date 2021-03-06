# [gulp](https://www.gulpjs.com.cn/)
### 前言
>  gulp是基于Nodejs的自动任务运行器，它能自动化地完成javascript/sass/less/html/image/css 等文件的的测试、检查、合并、压缩、格式化、浏览器自动刷新、部署文件生成，并监听文件在改动后重复指定的这些步骤。（[引用来自Kasmine的博客](https://www.cnblogs.com/kasmine/p/6436131.html)）。

> 使用Gulp的优势就是利用流的方式进行文件的处理，使用管道（pipe）思想，前一级的输出，直接变成后一级的输入，通过管道将多个任务和操作连接起来，因此只有一次I/O的过程，流程更清晰，更纯粹。Gulp去除了中间文件，只将最后的输出写入磁盘，整个过程因此变得更快（[引用来自Kasmine的博客](https://www.cnblogs.com/kasmine/p/6436131.html)）。

> 说起gulp我们都会谈到一个名词，便是前端自动化构建，接下来就前端自动化构建进行解释。

### 前端自动化构建
> "说到构建工具，我往往会在前面加「自动化」三个字，因为构建工具就是用来让我们不再做机械重复的事情，解放我们的双手的。"（[引用来自知乎Jasin Yip的回答](https://www.zhihu.com/question/35595198)）。

### gulp的安装及基本使用
#### 0. 安装 node.js 及 cnpm 淘宝镜像：
> 因为gulp是是基于Nodejs的自动任务运行器，所以必须安装[node.js](http://nodejs.cn/)。

>  由于npm是国外的，使用起来比较慢，我们这里使用淘宝的cnpm镜像来安装插件及应用，[cnpm安装指南](https://npm.taobao.org/)。

#### 1. 全局安装 gulp：
```
$ npm install --global gulp
```
#### 2. 作为项目的开发依赖（devDependencies）安装：
```
$ npm install --save-dev gulp
```
#### 3. 在项目根目录下创建一个名为 gulpfile.js 的文件：
```js
var gulp = require('gulp');

gulp.task('default', function() {
  // 将你的默认的任务代码放在这
});
```
#### 4. 运行 gulp
```
$ gulp
```
> 默认的名为 default 的任务（task）将会被运行，在这里，这个任务并未做任何事情。

### gulp的插件使用
> 相关插件用法请看对应链接，感谢原作者对插件的解析，如有侵权请联系删除谢谢。

> 按需配置好package.json 在根目录npm install即可。
- [node stream(流)](http://nodejs.cn/api/stream.html)
- [gulp-sourcemaps（显示编译前位置）的用法](https://www.cnblogs.com/zy20160429/p/8205025.html)
- [gulp-autoprefixer（根据设置浏览器版本自动处理css属性的浏览器前缀）的用法](https://www.npmjs.com/package/gulp-autoprefixer)
- [gulp-less（less编译）的用法及相关插件](https://www.npmjs.com/package/gulp-less)
- [gulp-sass（sass编译）的用法及相关插件](https://www.npmjs.com/package/gulp-sass)
- [gulp-clean-css(压缩CSS文件)的用法及相关插件](https://blog.csdn.net/gongm24/article/details/58696676)
- [gulp-uglify（js压缩插件）的用法及相关插件](https://www.npmjs.com/package/gulp-uglify)
- [gulp-rename(文件更名)的用法及相关插件](https://blog.csdn.net/gongm24/article/details/59079775)
- [gulp-concat(js文件合并)的用法及相关插件](https://www.npmjs.com/package/gulp-concat)
- [gulp-imagemin(图片压缩)的用法](https://www.npmjs.com/package/gulp-imagemin)
- [gulp-changed(只通过更改过的文件)的用法](https://www.npmjs.com/package/gulp-imagemin)
- [browsersync(浏览器实时刷新)的用法](http://www.browsersync.cn/docs/gulp/)
- [babel(es6 编译 es5)的用法](https://www.npmjs.com/package/gulp-babel)

> 下面附上我自己的项目gulpfile的文件源码

```js
// gulp引入
let gulp  = require('gulp');

// 路径
let SRC 		= './src',// 入口文件夹
	DIST 		= './static',// 出口文件夹
	LESS_SRC 	= SRC + '/**/*.less', // less编译入口 (** 匹配js文件夹的0个或多个子文件夹)  
	JS_SRC 		= SRC + '/**/*.js', // JS编译入口 (* 匹配js文件夹下所有.js格式的文件)
	IMG_SRC 	= SRC + '/**/images/*.{png,jpg,gif,ico}',// 图片编译入口
	HTML		= './*.html';// html文件

//插件
let less 		= require('gulp-less'),// less编译
	sourcemaps 	= require('gulp-sourcemaps'),// 浏览器显示编译前less或js位置
	uglify 		= require('gulp-uglify'),// 压缩js文件
	pump 		= require('pump'),// 可以使我们更容易找到代码出错位置
	cleanCSS 	= require('gulp-clean-css'),// css压缩
	rename 		= require("gulp-rename"),// 文件更名
	autoprefixer= require('gulp-autoprefixer'),// 根据设置浏览器版本自动处理css属性的浏览器前缀
	concat 		= require('gulp-concat'),// 合并javascript文件，减少网络请求。
	imagemin 	= require('gulp-imagemin'),// 图片压缩
	pngquant 	= require('imagemin-pngquant'), // 深度压缩  
	changed 	= require('gulp-changed'),// 只通过更改过的文件
	browserSync = require('browser-sync').create(),//浏览器实时刷新  
	babel 		= require('gulp-babel');// es6 编译 es5

// 刷新浏览器
let	reload      = browserSync.reload;

// less编译
gulp.task('less',  ()=> {
	let options = {  
        browsers: ['last 5 versions', 'Android >= 4.0'],// 浏览器版本
        cascade: true,//是否美化属性值 默认：true 像这样：
            //-webkit-transform: rotate(45deg);
            //        transform: rotate(45deg);
        remove:true //是否去掉不必要的前缀 默认：true 
    };  
  return gulp.src([LESS_SRC,'!'+SRC+'/common/**/*.less'])// 除了common文件夹下的less全部编译
  	.pipe(sourcemaps.init())// 启用sourcemaps功能
  	.pipe(changed(DIST))// 只通过更改过的文件
    .pipe(less())// 执行less编译操作
    .pipe(autoprefixer(options))
    .pipe(cleanCSS({
    	keepSpecialComments: '*'
        //保留所有特殊前缀 当你用autoprefixer生成的浏览器前缀，如果不加这个参数，有可能将会删除你的部分前缀
    }))// 开启css压缩
    .pipe(rename(
    	// {
	    // 	dirname: "css",// 生成到对应目录下，可以修改上级目录名字，但是所有只能生成到一个文件夹
		   //  basename: "aloha",// 文件名字
		   //  prefix: "bonjour-",// 文件前缀
		   //  suffix: ".min",// 文件后缀
		   //  extname: ".css"// 文件类型	    
    	// }
    	function (path) {
    		// path.dirname += "/css";// 生成到对应目录下，可以修改上级目录名字，但是所有只能生成到一个文件夹
		    // path.basename += "-goodbye";// 文件名字
		    // path.extname = ".md";// 文件类型	 
		    // 前缀后缀不生效
            path.dirname = path.dirname.replace('less', 'css');// 修改上级目录，可生成到对应目录
        }
    ))// 改名字
    .pipe(sourcemaps.write('./map'))// 生成记录位置信息的sourcemaps文件，为空时即为解析目录下
    .pipe(gulp.dest(DIST))// 编译出口，可生成目录或为存在目录
    .pipe(reload({stream: true}));
});

// js操作
gulp.task('js',  (cb)=> {
	// pump是一个小节点模块，将流连接在一起，如果其中一个关闭，则会将它们全部破坏
	// 当dest发出关闭或错误时，使用标准source.pipe（dest）事件源将不会被销毁。 
	// 您还不能提供回调来告诉管道何时完成。
	pump([
			gulp.src([JS_SRC]),// JS编译入口
			sourcemaps.init(),// 启用sourcemaps功能
			changed(DIST),// 只通过更改过的文件
			babel({
	            presets: ['env']// 编译成es5
	        }),
			uglify(),// 开启压缩功能
			concat({
				path:'js/all.js'
			}),// 开启js合并功能
			rename({suffix: '.min'}),// 改名字中间加‘.min’
			sourcemaps.write('./map'),// 生成记录位置信息的sourcemaps文件，为空时即为解析目录下
			gulp.dest(DIST),// 编译出口，可生成目录或为存在目录
			reload({stream: true})
		],cb// 函数有参数
	)
});

// 图片压缩（由于可能过多图片，不建议每次更新操作）
// cmd命令：gulp images
gulp.task('images',  ()=> {
		gulp.src([IMG_SRC])// JS编译入口
		.pipe(changed(DIST))// 只通过更改过的文件
		.pipe(imagemin(
			{
	            optimizationLevel: 5, //类型：Number  默认：3  取值范围：0-7（优化等级）
	            progressive: true, //类型：Boolean 默认：false 无损压缩jpg图片
	            interlaced: true, //类型：Boolean 默认：false 隔行扫描gif进行渲染
	            multipass: true, //类型：Boolean 默认：false 多次优化svg直到完全优化
	            svgoPlugins: [{removeViewBox: false}], // 不移除svg的viewbox属性  
            	use: [pngquant()] // 使用pngquant插件进行深度压缩
	        }
		))
		.pipe(gulp.dest(DIST))// 编译出口，可生成目录或为存在目录
	}
)

// 静态服务器 + 监听 less/js/html 文件
gulp.task('server',  ()=> {
	browserSync.init({
        server: {
            baseDir: "./"
        }
    });
});

// 执行默认操作，顺序执行另外的操作
gulp.task('default',['server'],()=> {
	// 执行操作
	gulp.start(['less','js']);
	// 监听改变
	gulp.watch(LESS_SRC, ['less']);			// 监听less的改变并执行对应操作
    gulp.watch(JS_SRC, ['js']);				// 监听js的改变并执行对应操作
    gulp.watch(HTML, reload);	
})
```
> 附上对应的package.json。
```
{
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-preset-env": "^1.6.1",
    "browser-sync": "^2.23.6",
    "gulp": "^3.9.1", 
    "gulp-autoprefixer": "^5.0.0",
    "gulp-babel": "^7.0.1",
    "gulp-changed": "^3.2.0",
    "gulp-clean-css": "^3.9.3",
    "gulp-concat": "^2.6.1",
    "gulp-imagemin": "^4.1.0",
    "gulp-less": "^4.0.0",
    "gulp-uglify": "^3.0.0",
    "imagemin-pngquant":"^5.1.0",
    "pump": "^3.0.0"
  }
}
```

### 问题记录
- gulp编译less，dirname.replace文件夹为css文件夹后，对应后缀如何添加.min？(gulp-rename)
