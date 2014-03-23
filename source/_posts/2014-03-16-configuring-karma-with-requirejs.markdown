---
layout: post
title: "Configuring Karma with Requirejs"
date: 2014-03-16 15:45:57 +0200
comments: true
published: false
categories: [requirejs, karma, unit test, javascript]
---
First we need to install `karma`.
For that you need `nodejs`. So if it is not installed go to http://nodejs.org and just click install.
Once `nodejs` is there you have access to its package manager `npm`

If you are on windows like me, and have Visual Studio 2013 installed you may want to tell npm to run the build using a specific version of Visual Studio.
You can do it on a per module installation basis
```bat Specify which version of Visual Studio to use when compiling sources for a module
npm install <module> --msvs_version=2013
``` 
or globally
```bat Globally set the version of Visual Studio for compiling sources
npm config set msvs_version 2013 --global
```

You can then install the karma command line
```bat Install karam cli
npm install -g karma-cli
```

and then once you are in your project
```bat Install karma for your project
npm install karma
karma init
```

Let me know what you think and happy coding.

Avi Haiat