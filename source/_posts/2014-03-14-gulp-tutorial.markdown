---
layout: post
title: "Gulp tutorial"
date: 2014-03-14 01:22:29 +0200
comments: true
published: false
categories: [gulp, deployment, grunt, automation]
---
There is a new frontend build tool out there, and it is making some noise - gulp -.
I thought I'd take a look and see if I could write a simple tutorial for Gulp.
This post will take you from installation and getting set up through examples of real world build processes.

```javascript Hello world gruntfile.js
var gulp = require('gulp');

gulp.task('default', function() {
 console.log('Hello world.');
});
```


```javascript copying file
var gulp = require('gulp');
 
var paths = {
 scripts: ['app/scripts/**/*.js'],
 html: ['app/index.html', '!app/test.html'], 
 dist: ['dist/']
};
 
gulp.task('default', function(){
 gulp.src(paths.scripts.concat(paths.html))
 .pipe(gulp.dest(paths.dist));
});
```