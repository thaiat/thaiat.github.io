---
layout: post
title: "How to install and use Octopress on Windows"
date: 2014-03-13 14:33:36 +0200
comments: true
categories: [git, github, github pages, octopress, blog]
---
A quick and easy 6 steps tutorial to get up and running with octopress and github pages

### Step 1: Install Chocolatey
1. Got to http://chocolatey.org
2. Open a command prompt
3. cd to `c:\`
4. Paste in and execute the following code from chocolatey home page
```bat Install Chocolatey
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex 
((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" 
&& SET PATH=%PATH%;%systemdrive%\chocolatey\bin
```

This will install chocolatey in `C:\Chocolatey`

### Step 2: Make sure Git is installed or install it via Chocolatey
```bat Install Git with Chocolatey
choco install git
```

To check that git is installed close and reopen a command prompt and run the following command:
```bat Check Git version
git --version
```

You should get `git version 1.0.0.msysgit.0`

### Step 3 : Install Ruby
1. Install `Ruby 1.9.3-p545` from `http://rubyinstaller.org`
2. Ruby should be installed in `C:\Ruby193`
3. Add the following path to the system variables Path : `c:\Ruby193\bin`
4. Exit and reopen command prompt and run `ruby --version`, you should get `ruby 1.9.3-p545`
5. Install ruby developpement kit with chocolatey

```bat Install Ruby Debkit with Chocolatey
c:
choco install ruby.devkit
```

This will install the devkit in `C:\Devkit`

### Step 4 : Install Octopress
Navigate to your github local repositories folder (for example mine is d:\github)   
The installation of Octopress consist in cloning the repository of Octopress
```bat Clone Octopress repository
git clone git://github.com/imathis/octopress.git blog
```

Navigate to the created folder and then run the following command to install the dependencies
```bat Install Octopress dependencies
gem install bundler
bundle install
```

Then install the default Octopress theme 
```bat Install Octopress default theme
rake install
```

Next edit `_config.yml` and adjust it with your blog properties (title,name, google tracker, etc...)

And now we can generate the site
```bat Generate and preview Octopress blog
rake generate && rake preview
```
You can open a browser and navigate to `http://localhost:4000`

### Step 5 : Blogging
To create a new post 
```bat Create a new blog post
rake new_post["Hello World Post No 1"]
```
The new post file is a simple markdown file generated in `source\_posts`

### Step 6 : Publishing your blog to github pages

Now you have no excuse not making a blog of your own...   
Happy Coding   
Avi