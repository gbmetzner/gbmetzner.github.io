---
layout: post
title: Multiple DB using Play Framework and Scala
---

Some time ago I had to change the database during a request depending on the logged user. This requirement looks somewhat weird but it's needed because our clients want their own database instance. So, we have a main application that manage logins and profiles and redirect the logged users to the right application's instance.

In this post I'm going to show you how I solved this problem, sharing the same application with a specific database instance based on the logged user. Problably there is a better way to solve it but this one was enough for me.

We're going to use [Play Framework](https://www.playframework.com/), [Scala](http://www.scala-lang.org), and [MySQL](http://www.mysql.com). 

First of all, we'll create a new Play Scala. I like to use [TypeSafe Activator](https://typesafe.com/activator) to create a new Scala projects.

To create a new Scala project type on your terminal:

> activator new

Typing it on terminal will show the list below:

[<img src="{{ site.baseurl }}/images/multipledb/01.png" style="width: 400px;"/>]

Choose number 6 to create a Play Scala project.



