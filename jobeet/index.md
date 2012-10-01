Practical Symfony: Day 1
========================

### Introduction
The Symfony2 has been an Open-Source project since 2005 and has become one of the most popular PHP frameworks thanks to its great features and great documentation. And this grand tradition started early on.

In December 2005, just after the first official release of symfony, we published the "Askeet tutorial", a set of 24 tutorials, published day-by-day between December 1st and Christmas.

This tutorial has proven to be an invaluable tool to promote the framework to newcomers. A lot of developers learned symfony thanks to askeet, and many companies still use askeet as their main training material.

But the askeet tutorial started to show its age and with the release of symfony 1.2, we decided to publish yet another advent calendar, Jobeet for Symfony2


### The Challenge
Each chapter/day is meant to last about one hour, and will be the occasion to learn Symfony2 by coding a real website, from start to finish.

One hour times twenty-four equals a day, and that's exactly how long we think that a developer needs to learn the fundamentals of Symfony2. Every day, new features will be added to the application, and we'll take advantage of this development to introduce you to new Symfony2 functionalities as well as good practices in Symfony2 web development.

For askeet, the 21st day was the "get-a-symfony-guru-for-a-day". We had no plan, and the community had to propose a feature to add to askeet. It was a great success and the community decided that we needed a search engine to the application. And we did it. The 21st day tutorial also proved to be one of the most popular of the askeet tutorials.


### This turorial is different
Remember the early days of PHP4. PHP was one of the first languages dedicated to the web and one of the easiest to learn.

But as web technologies evolve at a very fast pace, web developers need to keep up with the latest best practices and tools. The best way to learn is of course by reading blogs, tutorials, and books. We have read a lot of these, be they written for PHP, Python, Java, Ruby, or Perl, and many of them fall short when the author starts giving snippets of codes as examples.

You are probably used to reading warnings like:

"For a real application, don't forget to add validation and proper error handling."

or

"Security is left as an exercise to the reader."

or

"You will of course need to write tests."

What? These things are serious business. They are perhaps the most important part of any piece of code. And as a reader, you are left alone. Without these concerns taken into account, the examples are much less useful. You cannot use them as a good starting point. That's bad! Why? Because security, validation, error handling, and tests, just to name a few, take care to code right.

In this tutorial, you will never see statements like those as we will write tests, error handling, validation code, and be sure we develop a secure application. That's because Symfony2 is about code, but also about best practices and how to develop professional applications for the enterprise. We will be able to afford this luxury because Symfony2 provides all the tools needed to code these aspects easily without writing too much code.

Validation, error handling, security, and tests are first-class citizens in Symfony2, so it won't take us too long to explain. This is just one of many reasons why to use a framework for "real life" projects.

All the code you will read in this tutorial is code you could use for a real project. We encourage you to copy and paste snippets of code or steal whole chunks.


### The project
The application to be designed could have been yet another blog engine. But we want to use Symfony2 on a useful project. The goal is to demonstrate that Symfony2 can be used to develop professional applications with style and little effort.

We will keep the content of the project secret for another day as we already have much to do today. However, you already know the name of the application: Jobeet.


### What for today?
As 24 hours is plenty of time to develop an application with Symfony2, we won't write PHP code today. But even without writing a single line of code, you will start understanding the benefits of using a framework like Symfony2, just by bootstrapping a new project.

The objective of the day is to setup the development environment and display a page of the application in a web browser. This includes installation of Symfony2, creation of an application, and web server configuration.

### Prerequisites
First of all, check that you already have a working web development environment with a web server (Apache for example), a database engine (MySQL, PostgreSQL, or SQLite), and PHP 5.4 or later.

As we will use the command line a lot, it's better to use a Unix-like OS, but if you run a Windows system, it will also work fine, you'll just have to type a few commands in the cmd prompt.

As this tutorial will mostly focus on Symfony2, we will assume that you already have a solid knowledge of PHP 5 and Object Oriented programming.

### Symfony2 installation
The symfony community maintains some great documentation available at the [symfony2 website](http://symfony.com/doc/current/index.html "Documentation"). We will be following the installation process detailed in the symfony [book](http://symfony.com/doc/current/book/installation.html).

* Step 1: Setup our working directory:
```
cd ~/
```

* Step 2: Get Composer:
```
curl -s https://getcomposer.org/installer | php
```
  [Composer](http://getcomposer.org) is a PHP dependancy manager for PHP, we will be using it to download symfony, as well as, speed up our development by reusing PHP libraries that others have written.

* Step 3: Install Symfony2 standard edition
```
php composer.phar create-project symfony/framework-standard-edition ~/jobeet 2.1.x-dev
cd ~/jobeet
```

* Step 4: Launch Symfony
  PHP 5.4 includes it's own webserver that we will be using to develop the website.  
```
cd ~/jobeet/web
php -S localhost:8000
```
  Now you can open any browser to http://localhost:8000/config.php.  This URL will let you know of any problems that you might need to correct before you can run the Symfony2 standard edition.

### Version control
It is a good practice to use source version control when developing a web application. Using a source version control allows us to:

* work with confidence
* revert to a previous version if a change breaks something
* allow more than one person to work efficiently on the project
* have access to all the successive versions of the application
* In this section, we will describe how to use [Git with Symfony2](http://symfony.com/doc/current/cookbook/workflow/new_project_git.html). If you use another source code control tool, it must be quite easy to adapt what we describe for Git.


```
echo "/web/bundles/
/app/bootstrap*
/app/cache/*
/app/logs/*
/vendor/
/app/config/parameters.yml" > .gitignore

git init
git add .
git commit -m 'Initial Jobeet commit'

```


===See you Tomorrow
Well, time is over for today! Even if we have not yet started talking about Symfony2, we have setup a solid development environment, we have talked about web development best practices, and we are ready to start coding.

Tomorrow, we will reveal what the application will do and talk about the requirements we need to implement during the tutorial.

If you want to download the code for today, or any other day, it is available on a day-to-day basis in the official [Jobeet Git repository](https://github.com/mykehsd/jobeet)



