---
layout: post
title: Multiple DB using Play Framework and Scala
---

Some time ago I had to change the database during a request depending on the logged user. This requirement looks somewhat weird but it's needed because our clients want their own database instance. So, we have a main application that manage logins and profiles and redirect the logged users to the right application's instance.

In this post I'm going to show you how I solved it, sharing the same application with a specific database instance based on the logged user. Problably there is a better way to solve it but this one was enough for me.

We're going to use [Play Framework](https://www.playframework.com/), [Scala](http://www.scala-lang.org), [Slick](http://slick.typesafe.com), and [MySQL](http://www.mysql.com). 

### Step 1: Creating a Play application

First of all, we'll create a new Play Scala. I like to use [TypeSafe Activator](https://typesafe.com/activator) to create a new Scala projects.

To create a new Scala project type on your terminal:

> activator new

Typing it on terminal will show the list below:

<img src="{{ site.baseurl }}/images/multipledb/01.png" style="width: 400px;"/>

Choose the number 6 to create a Play Scala project and give a name to it, e.g *multidb* as below:

<img src="{{ site.baseurl }}/images/multipledb/02.png" style="width: 400px;"/>

### Step 2: Configuring Web Application and Database

After that, we're going to configure our web application and database.

Firstly, we need to configure two schemas on MySQL, something like this:

{% highlight sql %}
CREATE SCHEMA `user1` DEFAULT CHARACTER SET utf8 ;
{% endhighlight %}

and

{% highlight sql %}
CREATE SCHEMA `user2` DEFAULT CHARACTER SET utf8 ;
{% endhighlight %}


