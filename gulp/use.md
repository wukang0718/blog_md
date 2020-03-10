# 使用gulp打包传统项目
 传统的开发项目，基于**jquery**和**bootstrap**框架做的开发

 
 现在需要对项目做压缩处理，但是不能修改文件的相对位置
 
 创建一个项目将项目源码放在src目录下，执行
 ```
	npm init
 ```
 
 新建一个gulpfile.js,这是gulp运行需要的文件,所有的gulp的任务都是在这个文件里执行的  
 > 安装gulp依赖（gulp和cli）
 ```
	npm install gulp gulp-cli--save
 ```
 
 分析：  
	&emsp; 1. 首先获取到src的所有源文件，将需要编译压缩的文件分类  
	&emsp;&emsp;		js/css/image/html/others,其中的others不需要编译压缩，需要直接复制  
	&emsp; 2. 对每类文件做对应的任务  
	&emsp;  3. 执行任务（开始执行压缩任务之前，需要先清空dist“输出目录”）
	
> 1. 首先获取到src的所有源文件，将需要编译压缩的文件分类  

	``` javascript
		const fs = require("fs");
		const path = require("path");
		const srcPath = path.resolve("./src");
		const pluginPath = path.resolve("./src/javascript/plugin");
		const getSrcFile = () => {
			let fileClass = {
				js: [],
				css: [],
				html: [],
				image: [],
				others: []
			};

			/**
			 * 获取所有文件的
			 * @param filePath 文件夹路径
			 * @param status 是否需要打包
			 */
			function getAllFile(filePath, status) {
				let files = fs.readdirSync(filePath);
				files.forEach(file => {
					// 读取所有的文件夹
					let fileDir = path.join(filePath, file);
					let fileStat = fs.statSync(fileDir);

					if (fileStat.isDirectory()) {
						// 如果是文件夹 递归获取下面的文件
						// plugin文件夹下不打包
						if (fileDir === pluginPath) {
							getAllFile(fileDir, false);
						} else {
							getAllFile(fileDir, status);
						}
					} else if (fileStat.isFile()) {
						if (status) {
							// 需要打包的
							if (/\.js$/.test(file)) {
								fileClass.js.push(fileDir);
							} else if (/\.css$/.test(file)) {
								fileClass.css.push(fileDir);
							} else if (/\.html$/.test(file)) {
								fileClass.html.push(fileDir);
							}  else if (/\.(jpg|bmp|gif|ico|pcx|jpeg|tif|png|raw|tga)$/.test(file)) {
								fileClass.image.push(fileDir);
							} else if (!/\.less$/.test(file)) {
								// less文件不需要打包
								fileClass.others.push(fileDir);
							}
						} else {
							fileClass.others.push(fileDir);
						}
					}

				});
			}
			getAllFile(srcPath, true);
			return fileClass;
		};

		const {js, css, html, image, others} = getSrcFile();
	```
	
> 2. 对每类文件做对应的任务   

```javascript
	const distPath = path.resolve("./dist");
	const gulp  = require("gulp");
	const uglify = require("gulp-uglify");
	const babel = require('gulp-babel');
	const minifyCSS = require('gulp-minify-css');
	const imagemin = require('gulp-imagemin');
	const htmlmin = require('gulp-htmlmin');
	/**
	 * 获取文件输出路径
	 * @param filePath
	 * @returns {string}
	 */
	const getOutputPath = (filePath) => {
		let destPath = filePath.replace(srcPath, distPath);
		destPath = destPath.slice(0, destPath.lastIndexOf("\\"));
		return destPath;
	};

	/**
	 * 编译，打包压缩js
	 */
	function jsGulp (cb) {
		js.forEach(file => {
			jsTask(file);
		});
		cb();
	}

	function jsTask(file) {
		let destPath = getOutputPath(file);
		return gulp.src(file)
			.pipe(babel({
				presets: ['@babel/env']
			}))
			.pipe(uglify())
			.pipe(gulp.dest(destPath))
	}

	/**
	 * 压缩处理css
	 */

	function cssGulp(cb) {
		css.forEach(file => {
			cssTask(file);
		});
		cb();
	}

	function cssTask(file) {
		let destPath = getOutputPath(file);
		return gulp.src(file)
			.pipe(minifyCSS())
			.pipe(gulp.dest(destPath))
	}

	/**
	 * 压缩图片 ----------- 依赖安装失败（暂不启用压缩）
	 */
	function imageGulp(cb) {
		image.forEach(file => {
			imageTask(file);
		});
		cb();
	}

	function imageTask(file) {
		let destPath = getOutputPath(file);
		return gulp.src(file)
			// TODO 图片压缩安装失败
			.pipe(imagemin({
				progressive: true
			}))
			.pipe(gulp.dest(destPath));
	}


	/**
	 * 压缩html
	 */
	function htmlGulp(cb) {
		html.forEach(file => {
			htmlTask(file);
		});
		cb();
	}

	function htmlTask(file) {
		const htmlOptions = {
			keepClosingSlash: true,
			caseSensitive: true,
			removeComments: true,//清除HTML注释
			collapseWhitespace: true,//压缩HTML
			removeEmptyAttributes: true,//删除所有空格作属性值 <input id="" /> ==> <input />
			removeScriptTypeAttributes: true,//删除<script>的type="text/javascript"
			removeStyleLinkTypeAttributes: true,//删除<style>和<link>的type="text/css"
			minifyCSS: true,
			minifyJS: true,
			continueOnParseError: true
		};
		let destPath = getOutputPath(file);
		return gulp.src(file)
			// TODO  pre标签中的for循环中出现 < 压缩失败
			.pipe(htmlmin(htmlOptions))
			.pipe(gulp.dest(destPath));
	}

	/**
	 * 复制其他格式的文件
	 */

	function copyGulp(cb) {
		others.forEach(file => {
			copyTask(file);
		});
		cb();
	}

	function copyTask(file) {
		let destPath = getOutputPath(file);
		return gulp.src(file)
			.pipe(gulp.dest(destPath))
	}
```
> 3. 执行任务（开始执行压缩任务之前，需要先清空dist“输出目录”）  
 
```javascript
	/**
	 * 删除dist文件夹
	 */
	function cleanGulp(cb) {
		del.sync("dist");
		cb()
	}
	// series 串行执行所有任务
	// parallel 并行执行所有任务
	// 先删除dist文件夹，在执行打包任务（所有打包任务一起执行）
	exports.default = gulp.series(cleanGulp, gulp.parallel(jsGulp, cssGulp, htmlGulp, imageGulp, copyGulp));
```

> pachage.json文件（配置scripts，安装所需的所有依赖）

```json
	{
	  "name": "myProject",
	  "version": "1.0.0",
	  "description": "",
	  "main": "gulpfile.js",
	  "scripts": {
		"build": "gulp",
		"test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "author": "",
	  "license": "ISC",
	  "dependencies": {
		"@babel/core": "^7.8.7",
		"@babel/preset-env": "^7.8.7",
		"del": "^5.1.0",
		"gulp": "^4.0.2",
		"gulp-babel": "^8.0.0",
		"gulp-cli": "^2.2.0",
		"gulp-htmlmin": "^5.0.1",
		"gulp-imagemin": "^7.1.0",
		"gulp-minify-css": "^1.2.4",
		"gulp-uglify": "^3.0.2",
		"html-minifier": "^4.0.0",
		"optipng": "^2.1.0"
	  }
	}
```
> 运行打包命令
```
	npm run build
```
* **dist目录就是打包压缩后的项目**（文件位置和名称没有变化）  
* **如果打包过程这个出现了图片压缩失败的问题，建议删除node_modules重新执行npm install安装依赖**
 