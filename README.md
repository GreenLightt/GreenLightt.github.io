

# 压缩处理

**前置要求**

```
$ npm install gulp -g
$ npm install gulp-minify-css gulp-uglify gulp-htmlmin gulp-htmlclean gulp --save
```

**压缩资源**

执行以下命令， 就会根据 `gulpfile.js` 中的配置，对 `public` 目录中的静态资源文件进行压缩。
```
hexo g && gulp
```
