---
layout: post
title: "How to install and use Octopress on Windows"
date: 2014-03-13 14:33:36 +0200
comments: true
categories: [git, github, github pages, octopress, blog]
---
A quick and easy 6 steps tutorial to get up and running with octopress and github pages

>**NOTE:**   
> As a pre-requisite you should make sure you have a public github repo with the name : `myusername.github.io`

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
Github pages expect to have two branches, the source branch and the master branch. The master branch is the one that github pages will show. The changes in the source branch won’t be published until you push your changes to master. You can configure those branchs manually and add your github pages repository as a remote, but there is also a rake task that does all for you: 
```bat rake command for setting up github pages
rake setup_github_pages[repo].
```
In the case of this blog, I ran `rake setup_github_pages[git@github.com:myusername/myusername.github.io.git]` and then `rake deploy` to upload the blog.   
In a snap the blog will be accessible at the url http://myusername.github.io.

Remember that rake deploy just generates the blog a push to the master branch. Your source branch won’t be uploaded to github if you don’t want to. You probably want, to have a secure backup online, among other reason. Commit your changes and do 
```bat pushing the sources
git push origin source.
```


##Workflow when editing your blog
1. Make your modification to your local repo using `rake new_post` and editing your markdown
2. `rake generate && rake preview` to check your changes
2. `git add .` and `git commit -m "update blog"`
3.  `rake deploy` to deploy the resulting blog
4. `git push origin source` to deploy your Octopress sources


Now you have no excuse not making a blog of your own...   
Happy Coding   
Avi