---
layout: post
title: "Angularjs and Requirejs for very large applications"
date: 2014-02-26 10:14:06 +0200
comments: true
categories: [angularjs, architecture, requirejs, lazy loading]
---
>**NOTE:**   
>This article assumes that you are familiar with both `requirejs` and `angularjs`

Live demo is available here : [http://thaiat.github.io/directory-angular-ionic-require][2]


Hi all, 

I've been working with `angularjs` for the past 3 months.

I like very much the framework, but i had a few issues with what would be the best way to organize the code when writing a large application. Especially i didn't like editing the index.html each time i would write a new directive, controller or service. So i looked at `requirejs` and started to think how the 2 libraries would play together.

In a nutshell here is my attempt to combine both `angularjs` and `requirejs`.   
Some may argue that this is unnecessary, but i wanted to explain my approach and see what is the feedback.   
The main reason i needed that is that i'm still using Visual Studio as my editor (yes... i know...).
Running the unit test with jasmine in Visual Studio works without an html page, so i needed a way with pure javascript files to link together all my unit tests. `requirejs` does exactly that...
 
So, first, let's see the file structure.

##File and folder structure
As per the best practices in angular, i wanted to organize my code by features.   
So the folder structure looks like this:

```bat folder structure
index.html
scripts/
   main.js   
   app/    
      area1/   
      area2/   
   libs/   
      angular/   
      ionic/   
      require/   
styles/
   font-awesome/
   main.css    
```

At the root we have scripts and styles, nothing surprising there.
In the subfolder `scripts/libs`, i usually put all the external javascript libraries.   
I like to create a subfolder underneath which is named as the current version number of the library (x.y.z).

For example:
```bat folder structure
libs/
   angular/
      1.2.9/
           angular-animate.js
           angular-animate.min.js
           angular-cookies.js
           .....
```

This makes it super easy to try a new version of the library and see if it works, while falling back to the previous one if the new version generates too many bugs.

Right at the root we have the `index.html`.   
Here is the code:
```html index.html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <link href="styles/main.css" rel="stylesheet" />

        <title>Hello World</title>
   
    </head>

    <body>
    
        <script src="scripts/libs/require/2.1.10/require.js" data-main="scripts/main"> </script>
    </body>
</html>
```

As you can see it is also very simple.   
It references the needed css, and has only a single script tag, pointing to `requirejs` and defining `scripts/main.js` as the entry point for the application.

>**NOTE:**   
>Note that the `.js` extension is omitted when referencing a javascript file with `requirejs`

###main.js
This file first defines the libraries needed by the application. We will see later on how we start the application using require entry point function.

So for the moment the content is pretty simple:
```javascript scripts/main.js
require.config({
      paths : {
        'angular' : 'libs/angular/1.2.9/angular',
        'angularUiRouter' : 'libs/angular/angular-ui/0.2.8/angular-ui-router',
        'angularSanitize' : 'libs/angular/1.2.9/angular-sanitize',
        'angularAnimate' : 'libs/angular/1.2.9/angular-animate'

    },
    shim : {
        'angular' : {
            exports : 'angular'
        },
        'angularUiRouter' : {
            deps : ['angular']
        },
        'angularAnimate' : {
            deps : ['angular']
        },
        'angularSanitize' : {
            deps : ['angular']
        }

    }
});

```
###feature folder
For the purpose of the explanation let's call this feature folder `area`.   
Before we review how we can start the application, let's first see how a feature is organized.

For each feature we will define a folder containing the following:
####area/namespace.js
This file is a returning the namespace of the feature.
```javascript scripts/app/area/namespace.js
define([], function () {
    'use strict';
    return "area";
});
```

Note that I usually put in the app folder a root namespace looking like this:
```javascript scripts/app/namespace.js
define([], function () {
    return "myApp";
});
```

In the namespace.js of a specific feature i can require this top root namespace to chain the namespaces.
area/namespace.js becomes:
```javascript scripts/app/area/namespace.js
define([
    '../namespace'  // we are requiring here the top root namespace
], function (namespace) {
    'use strict';
    return namespace + ".area";
});
```

I find this elegant because the actual name of the namespace are only hard coded once. I could decide later on to change the namespace and this will not impact any other part of the application...

Next to namespace.js we define 2 others files : `module.js` and `module.require.js`.   
`module.js` simply defines a specific module for the feature:

```javascript scripts/app/area/module.js
define([
        'angular',
        './namespace'
],
    function (angular, namespace) {
        'use strict';

        return angular.module(namespace, []);

    });
```

See how we reference the namespace for the feature instead of duplicating the hard coded namespace.

`module.require.js` is the file responsible for binding together all the files we will put in our feature module (filters, directives, services, controllers, etc...)   
To achieve a clean separation between those files, we will create sub folders per nature inside the feature folder:

```bat flder structure
configs/
constants/
controllers/
directives/
filters/
services/
templates/
```

Each of these sub folders contains a file per component, plus an `_index.js` file referencing all of them.   
To better understand let see how the services folder looks like:
####area/services/_index.js
```javascript scripts/app/area/services/_index.js
define([
    './myService'
], function () {
    'use strict';
});
```

####area/services/myService.js
```javascript scripts/app/area/services/myService.js
define([
        '../module',
        '../namespace'
    ],
    function (module, namespace) {
        'use strict';

        var name = namespace + ".myService";
        var dependencies = ['$log'];
        var service = function ($log) {
            return {
                add : function (v1, v2) {
                    $log.debug("add function");
                    return v1 + v2;
                }

            };
        };

        module.factory(name, dependencies.concat(service));
    });
```

Note that we are again referencing the namespace in order to prefix the name of the service with it.   
If you do not want your service to be in a specific namespace you can just ignore that and write:
```javascript
var name = "myService"; // instead of var name = namespace + ".myService";
```

####area/module.require.js
This file simply references all the _index.js per component type

```javascript scripts/app/area/module.require.js
define([
    './configs/_index',
    './constants/_index',
    './controllers/_index',
    './directives/_index',
    './filters/_index',
    './services/_index'
], function () {
    'use strict';

});
```

Now lets starts our application.
We will need an app.js file in `scripts/app`
```javascript scripts/app/app.js
define([
        'angular',
        './namespace',
        './area/namespace',

        './area/module.require'
    ],
    function (angular, namespace, namespaceArea) {
        'use strict';

        var app = angular.module(namespace, [
            namespaceArea
        ]).run(function () {
            //
        });
        return app;
    });
```

We are requiring the feature `module.require.js` file to make sure all the files making the feature are loaded. And in addition we are also referencing the namespace of the application and the namespace of the feature in order to prevent hard coding again their values.

The only thing left is to bind `main.js` with `app.js` using the following code
at the end of `main.js`

```javascript scripts/main.js
require([
        'angular',
        'app/namespace',
        'app/app'
    ],
    function (angular, namespace) {
        angular.element(document).ready(function() {
             angular.bootstrap(document, [namespace]);
       });
    });
```

This may seem over complicated, but once you understand how it works it is really easy to expand.
Each time i need a new "feature", i'm copying a dummy feature structure, set the namespace, and reference the module.require of the feature in the main `app.js`. 
Whenever i need a new component inside the feature, i put it in the correct location based on its type, and edit the corresponding `_index.js`.



To better illustrate these concepts you can look at this [repository][1].   
Live demo is available here : [http://thaiat.github.io/directory-angular-ionic-require][2]

I have re-written the Sample Mobile Application of Christophe Coenraets using these technics (see [http://coenraets.org/blog/2014/02/sample-mobile-application-with-ionic-and-angularjs/][3])

Let me know what you think and happy coding.

Avi Haiat


  [1]: https://github.com/thaiat/directory-angular-ionic-require
  [2]: http://thaiat.github.io/directory-angular-ionic-require
  [3]: http://coenraets.org/blog/2014/02/sample-mobile-application-with-ionic-and-angularjs/