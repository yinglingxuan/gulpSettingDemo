# gulpSettingDemo
gulp常用的配置和演示
对应的插件
var gulp = require('gulp');
var concat = require('gulp-concat');  //合并文件，减少网络请求。
var header = require('gulp-header');
var notify = require("gulp-notify");  //显示报错信息和报错后不终止当前gulp任务。
var connect = require("gulp-connect"); //这个gulp-connect启动服务器（并能时时同步）。
var sass = require("gulp-sass"); 
var autoprefixer = require('gulp-autoprefixer');  //插件可以根据我们的设置帮助我们自动补全浏览器的前缀(如：-moz、-ms、-webkit、-o)
var webserver = require('gulp-webserver');//是开启服务器，通常和gulp-livereload结合使用
var livereload = require('gulp-livereload');  //配置热加载,网页刷新
var sourcemaps = require('gulp-sourcemaps');  //就是文件压缩后不利于查看与调试，但是有了sourcemap，出错的时候，除错工具将直接显示原始代码，而不是转换后的代码）
var plumber = require('gulp-plumber');    //可以阻止 gulp 插件发生错误导致进程退出并输出错误日志
var ext_replace = require('gulp-ext-replace');  //用于更改文件扩展名。
var cssmin = require('gulp-cssmin');
var uglify = require('gulp-uglify');  //压缩
var spritesmith = require("gulp.spritesmith")//配置雪碧图的插件



对应插件的使用

//运行错误后自动中断执行
var onError = function (err) {  //设置异常
    notify.onError({
        title: "Gulp",
        subtitle: "Failure!",
        message: "Error: <%= error.message %>",
        sound: "Beep"
    })(err);
    this.emit('end');
};
var scripts = [
    './src/lib/modal.js',
    './src/lib/hotcss.js'
    // './src/lib/star-rating.js'
];
//合并js插件
gulp.task('scripts', function () {
    return gulp.src(scripts)
        .pipe(concat('lib.js'))   //合并后的文件名
        .pipe(gulp.dest('dist/js')) //开两个为两个流
        // .pipe(uglify())
        .pipe(gulp.dest('dist/js')); //第二个下标的目录
});

//编译sass文件
gulp.task('sass', function () {
    return gulp.src(['./src/style/*.css'])
        .pipe(plumber({     //处理异常
            errorHandler: onError
        }))
        // .pipe(sass()) //把scss转为css
        // .pipe(sourcemaps.init())//生产时候启用，部署时取消
        .pipe(autoprefixer([   //浏览器的版本
            'last 2 versions',
            'ff >= 30',
            'chrome >= 34',
            'ios >= 6',
            'android >= 4.4',
            'and_uc 9.9',
			'IE >= 6' 
        ]))
        // .pipe(sourcemaps.write())
        // .pipe (cssmin())
        .pipe(concat('style.css'))
        .pipe(gulp.dest('./dist/css/'))
        .pipe(livereload());
});




// 合并js
gulp.task('customJs', function () {
    return gulp.src('./src/js/*.js')
        .pipe(plumber({
            errorHandler: onError
        }))
        .pipe(uglify()) //压缩文件，部署时启用
        .pipe(concat('custom.js'))
        .pipe(gulp.dest('./dist/js'));
});

var replace = require('gulp-replace');
var fs = require("fs");
//公共html部分
gulp.task('include', function () {
    var htmlDir = './dist/demos/';
    fs.readdir(htmlDir, function (err, files) {
        if (err) {
            console.log(err);
        } else {
            files.forEach(function (f) {
                if (f !== '_header.html' && f !== '_footer.html') {
                    gulp.src(htmlDir + f)
                        .pipe(replace(/<!--header-->[\s\S]*<!--headerend-->/, '<!--header-->\n' + fs.readFileSync(htmlDir + '_header.html', 'utf-8') + '\n<!--headerend-->'))
                        .pipe(replace(/<!--footer-->[\s\S]*<!--footerend-->/, '<!--footer-->\n' + fs.readFileSync(htmlDir + '_footer.html', 'utf-8') + '\n<!--footerend-->'))
                        .pipe(gulp.dest(htmlDir))
                }
            });
        }
    });
});


//搭建本地服务器
gulp.task('webserver', function () {
    gulp.src('./dist')
        .pipe(plumber({
            errorHandler: onError
        }))
        .pipe(webserver({
            port: 8124,
            livereload: true,
            directoryListing: false,
            open: true
        }));
});

//配置雪碧图
gulp.task('sprites', function () {
    return gulp.src('./src/imgs/*.png')//需要合并的图片地址
		.pipe(plumber({
			errorHandler: onError
		}))
        .pipe(spritesmith({
            imgName: 'demo.png',//保存合并后图片的地址
            cssName: './dist/css/sprites.css',//保存合并后对于css样式的地址
            padding:5,//合并时两个图片的间距
            algorithm: 'binary-tree',//注释1
            // cssTemplate:"css/handlebarsStr.css"//注释2
        }))
        .pipe(gulp.dest('./dist/images/'));
});

gulp.task("default", ['webserver', 'watch', 'sass', 'scripts', 'customJs','sprites']);

//监听项目 即时刷新
gulp.task('watch', function () {
    livereload.listen();  //监听页面刷新
    gulp.watch('./src/js/*.js', ['customJs']);  //刷新文件 watch 刷新的工具类
    gulp.watch('./src/lib/*.js', ['scripts']);
    gulp.watch('./src/style/*.css',['sass']);
    // gulp.watch(['./dist/demos/_header.html','./dist/demos/_footer.html'], ['include']);  //公共HTML
});
