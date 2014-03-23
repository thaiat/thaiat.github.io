---
layout: post
title: "Securing angularjs with Auth0"
date: 2014-03-14 14:14:41 +0200
comments: true
published: false
categories: [security, auth0, angularjs, openid, oauth2, identity]
---
>**NOTE:**   
> This articles assumes that you are familiar with both `angularjs` and `requirejs`, and that you have read my previous post []

## What is Auth0 ?
If you want to be serious about security in your application, I recommand you to check **Auth0** at [auth0.com][1].


Auth0 is amaaazing!!!!!
In a nutshell it is a a set of unified APIs and tools that instantly enables Single Sign On and user management for all your applications. It supports social identity providers as well as enterprise identity providers such as Active Directory, LDAP, Office365, Google Apps, Salesforce, and many more.

It is focused on the identity management meaning it is much more powerful than the built-in capabilities of the *Backend as a Service* offers (parse, firebase,etc...) .

It works virtually on any platform. It is simple to use, free to try, and the guys behind it are awesome.   
They have a [chat][2] where you can ask anything. Try it, you'll be impressed how reactive they are...

## Requirements
OK... Let's quickly review what we want to build.   
So our goal is to have an angular module, managing security and user identity.

List of requirements :

 * We want the service to use promises.
 
 * We want to control the look and feel of the authentication form.
 
 * We want it to play nicely with `cordova` or `phonegap`, meaning when we are poping up a new window for doing the `OAuth2` dance (navigating for example to google and then back to the application) we want to use the `InAppBrowser` plugin, but also fallback to standard `window.open` when testing with a desktop browser.
 
 * We don't want to use cookies because they are curse, and we are much better using tokens.
 

Auth0 already provides at least 3 javascripts components that we could use, but there are some caveats.

**The first one** is their standard javascript client available at [https://github.com/auth0/auth0.js][3]   
*Problem* : the script uses `browserify` which creates an incompatibility with `requirejs`, and use standard `xhr` calls instead of angular promises.

**The second one** is [https://github.com/auth0/auth0-angular][4]   
*Problem* : It's a good starting point, but... it uses internally the previous script , plus it has a dependency on `ngCookies`. Also it uses redirection instead of poping up a new window.

**The third one** is [https://github.com/auth0/phonegap-auth0][5]   
*Problem* : no fallback mechanism to `window.open` when testing with a desktop browser...

So let's rebuild something using these different pieces as a starting point.

The high level api of our authentication service api should be

Function | Description 
--- | --- 
login | Logs in the user
logout | Logs out the user
signup | Sign up a new user



##login function
The login function is versatile as we should be able to login with a username + password against what auth0 calls a local database (simple account hosted by auth0), but we also want to be able to use Social or Enterprise provider like google, office365, yahoo, etc...
In the latter case, we are doing passive authentication meaning the user before he types in its credentials is redirected to the external provider site, and then if successfully authenticated a callback url of our choice is called, and an access token is appended to the url.
Because we want to use the authentication both with Cordova and a standard desktop browser, we don't have

Let me know what you think and happy coding.

Avi Haiat


  [1]: https://auth0.com
  [2]: http://chat.auth0.com
  [3]: https://github.com/auth0/auth0.js
  [4]: https://github.com/auth0/auth0-angular
  [5]: https://github.com/auth0/phonegap-auth0