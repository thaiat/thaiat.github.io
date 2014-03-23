---
layout: post
title: "Gulp tutorial"
date: 2014-03-14 01:22:29 +0200
comments: true
published: false
categories: [gulp, deployment, grunt, automation]
---
There is a new frontend build tool out there, and it is making some noise - gulp -.
I thought I'd take a look and see if I could write a simple tutorial using gulp.
This post will take you from installing gulp and getting set up through examples of real world build processes.

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


```javascript
var gulp = require('gulp');
 
var clean = require('gulp-clean');
var jshint = require('gulp-jshint');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var imagemin = require('gulp-imagemin');
 
var bases = {
 app: 'app/',
 dist: 'dist/',
};
 
var paths = {
 scripts: ['scripts/**/*.js', '!scripts/libs/'],
 libs: ['scripts/libs/jquery/dist/jquery.js', 'scripts/libs/underscore/underscore.js', 'scripts/backbone/backbone.js'],
 styles: ['styles/**/*.css'],
 html: ['index.html', '404.html'],
 images: ['images/**/*.png'],
 extras: ['crossdomain.xml', 'humans.txt', 'manifest.appcache', 'robots.txt', 'favicon.ico'],
};
 
// Delete the dist directory
gulp.task('clean', function() {
 return gulp.src(bases.dist)
 .pipe(clean());
});
 
// Process scripts and concatenate them into one output file
gulp.task('scripts', function() {
 gulp.src(paths.scripts, {cwd: bases.app})
 .pipe(jshint())
 .pipe(jshint.reporter('default'))
 .pipe(uglify())
 .pipe(concat('app.min.js'))
 .pipe(gulp.dest(bases.dist + 'scripts/'));
});
 
// Imagemin images and ouput them in dist
gulp.task('imagemin', function() {
 gulp.src(paths.images)
 .pipe(imagemin())
 .pipe(gulp.dest(bases.dist));
});
 
// Copy all other files to dist directly
gulp.task('copy', function() {
 // Copy html
 gulp.src(paths.html, {cwd: bases.app})
 .pipe(gulp.dest(bases.dist));
 
 // Copy styles
 gulp.src(paths.styles, {cwd: bases.app})
 .pipe(gulp.dest(bases.dist + 'styles'));
 
 // Copy lib scripts, maintaining the original directory structure
 gulp.src(paths.libs, {cwd: 'app/**'})
 .pipe(gulp.dest(bases.dist));
 
 // Copy extra html5bp files
 gulp.src(paths.extras, {cwd: bases.app})
 .pipe(gulp.dest(bases.dist));
});
 
// A development task to run anytime a file changes
gulp.task('watch', function() {
 gulp.watch('app/**/*', ['scripts', 'copy']);
});
 
// Define the default task as a sequence of the above tasks
gulp.task('default', ['clean', 'scripts', 'imagemin ', 'copy']);
```

Let me know what you think and happy coding.

Avi Haiat