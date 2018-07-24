---
layout: post
title:  "Virtual environments for Python, Ruby and Java"
categories: [Coding, Virtual Environments]
tags: [coding, java, ruby, python]
date:   2018-07-22 08:44:21 -0700
---


I work with multiple languages and code mostly in Python, Ruby, Java or scala. When working on a project, I would like to isolate the dependencies within the project and not to mess up with the system libraries. I will go through three virtual environments in this post which can be handy while working on a project and gives the flexibility to switch between different versions of the language.

**virtualenv - for Python**  
[virtualenv](https://virtualenv.pypa.io/en/stable/) is a tool to create isolated Python environments.

{% highlight shell %}
# installing virtualenv
$ [sudo] pip install virtualenv

# initializing virtualenv
$ virtualenv venv # here venv is the folder name and this command creates venv folder

# activating virtualenv
$ source venv/bin/activate

#install dependencies within the folder
$ pip install requests # this will install request lib inside venv folder and you can import requests and run python code within (venv) context.

#deactivate current virtualenv context
$ (venv) deactivate

# creating virtualenv for a particular python version
$ virtualenv --python=/usr/bin/python2.6 venv
{% endhighlight %}

**rbenv - for Ruby**   
[rbenv](https://github.com/rbenv/rbenv) is a tool for simple ruby version management. Installation details can be found [here](https://github.com/rbenv/rbenv#installation)

Gist of all commands:
{% highlight shell %}
$ brew install rbenv # Installation for mac
$ rbenv init    # initialize rbenv
$ rbenv install -l  # list all available versions:
$ rbenv install 2.0.0-p247 # install a Ruby version
$ rbenv local 1.9.3-p327 # sets the local ruby version
$ rbenv local -unset #unset the local version
$ rbenv global 1.9.3-p327 # set the global system version of ruby
$ rbenv versions #list all ruby versions known to rbenv
$ rbenv shell 1.9.3 # set shell specific ruby.
{% endhighlight %}

**jEnv - for Java**  
[jEnv](http://www.jenv.be/) is a command line tool to manage java environment.

Commands:
{% highlight shell %}
$ brew cask install java #this will install java10
To install java 8, you can use the below commands
$ brew tap caskroom/versions
$ brew cask install java8

$ brew install jenv #install jenv in mac OS
$ jenv add /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home 
$ jenv add /Library/Java/JavaVirtualMachines/jdk17011.jdk/Contents/Home
{% endhighlight %}

You can check the folder /System/Library/Java/JavaVirtualMachines/ for the available versions to add to jenv

{% highlight shell %}
$ jenv versions #shows all the jenv added versions
$ jenv global oracle64-1.6.0.39 # Configure global version
$ jenv local oracle64-1.6.0.39 # Configure local version (per directory)
$ jenv shell oracle64-1.6.0.39 # Configure shell instance version
{% endhighlight %}

Hope the above virtual environments are useful while setting up your projects.