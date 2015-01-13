---
layout: post
title: Multiple DB using Play Framework and Scala
---

Some time ago was asked to me o choose the database during a request depending on the logged user. 

This requirement looks somewhat weird, but it's needed because our clients want their own database instance. 
So, we have a main application that manage logins and profiles and redirect the logged users to the right application's instance.

In this post I'm going to show you how I solved it, sharing the same application with a specific database instance based on the logged user. Problably there is a better way to solve it but this one was enough for me.

We're going to use [Play Framework](https://www.playframework.com/), [Scala](http://www.scala-lang.org), [Slick](http://slick.typesafe.com), and [MySQL](http://www.mysql.com). 

----

### Step 1: Creating a Play application

First of all, we'll create a new Play Scala. I like to use [TypeSafe Activator](https://typesafe.com/activator) to create a new Scala projects.

To create a new Scala project type on your terminal:

> activator new

Typing it on Terminal will show the ouput below:

`Fetching the latest list of templates...` <br/>

`Browse the list of templates: http://typesafe.com/activator/templates` <br/>
`Choose from these featured templates or enter a template name:` <br/>
  `1) minimal-akka-java-seed` <br/>
  `2) minimal-akka-scala-seed` <br/>
  `3) minimal-java` <br/>
  `4) minimal-scala` <br/>
  `5) play-java` <br/>
  `6) play-scala` <br/>
`(hit tab to see a list of all templates)` <br/>

Choose the number 6 to create a Play Scala project and give a name to it, e.g *multidb* as below:

`> 6` <br/>
`Enter a name for your application (just press enter for 'play-scala')` <br/>
`> multidb` <br/>
`OK, application "multidb" is being created using the "play-scala" template.` <br/>

`To run "multidb" from the command line, "cd multidb" then:` <br/>
`/Users/gbmetzner/Documents/Development/projects/blogs/multidb/activator run` <br/>

`To run the test for "multidb" from the command line, "cd multidb" then:` <br/>
`/Users/gbmetzner/Documents/Development/projects/blogs/multidb/activator test` <br/>

`To run the Activator UI for "multidb" from the command line, "cd multidb" then:` <br/>
`/Users/gbmetzner/Documents/Development/projects/blogs/multidb/activator ui` <br/>

`Gustavos-MacBook-Pro:blogs gbmetzner$` <br/>

----

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

We need to create tables, so, execute the code below in both schemas user1 and user2:

{% highlight sql %}
CREATE TABLE `user1`.`user` (
`id` INT NOT NULL AUTO_INCREMENT,
`value` VARCHAR(45) NOT NULL,
PRIMARY KEY (`id`));
{% endhighlight %}

Now our database is ready.

It's time to configure our web application.

Open the application.conf file located on /conf/application.conf and type the following to configure these data sources on Play:

`db.user1.driver = com.mysql.jdbc.Driver` <br/>
`db.user1.url = "jdbc:mysql://localhost:3306/user1"` <br/>
`db.user1.user = root` <br/>
`db.user1.password = "root"` <br/>

and

`db.user2.driver = com.mysql.jdbc.Driver` <br/>
`db.user2.url = "jdbc:mysql://localhost:3306/user2"` <br/>
`db.user2.user = root` <br/>
`db.user2.password = "root"` <br/>

----

### Step 3: Configuring Slick

Configure Slick is quite simple, actually I don't know if I'm doing the right thing but it's working pretty well for me.

Firstly, we need to configure our build.sbt to add Slick and MySQL connector as following:

{% highlight scala %}
libraryDependencies ++= Seq(
  jdbc,
  "com.typesafe.slick" %% "slick" % "latest.integration",
  "mysql" % "mysql-connector-java" % "latest.integration"
)
{% endhighlight %}

**Note**: Do not use "latest.integration" on production. I like to use it to keep my projects up-to-date while it's in dev mode.

After that, we need to update our project to download these libraries. To do that go to the root directory of our project
and type _activator_ and right after type _update_ as following:

`Gustavos-MacBook-Pro:multidb gbmetzner$ activator` <br/>
`Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=1G; support was removed in 8.0` <br/>
`[info] Loading global plugins from /Users/gbmetzner/.sbt/0.13/plugins` <br/>
`[info] Updating {file:/Users/gbmetzner/.sbt/0.13/plugins/}global-plugins...` <br/>
`[info] Resolving org.fusesource.jansi#jansi;1.4 ...` <br/>
`[info] Done updating.` <br/>
`[info] Loading project definition from /Users/gbmetzner/Documents/Development/projects/scala/multidb/project` <br/>
`[info] Updating {file:/Users/gbmetzner/Documents/Development/projects/scala/multidb/project/}multidb-build...` <br/>
`[info] Resolving org.fusesource.jansi#jansi;1.4 ...` <br/>
`[info] Done updating.` <br/>
`[info] Set current project to multidb (in build file:/Users/gbmetzner/Documents/Development/projects/scala/multidb/)` <br/>
`[multidb] $ update` <br/>
`[info] Updating {file:/Users/gbmetzner/Documents/Development/projects/scala/multidb/}root...` <br/>
`[info] Resolving jline#jline;2.12 ...` <br/>
`[info] Done updating.` <br/>
`[success] Total time: 6 s, completed 03/01/2015 18:26:52` <br/>
`[multidb] $ ` <br/>

Within the models package I created a case class called User as following:

{% highlight scala %}
package models

import scala.slick.driver.MySQLDriver.simple._
import scala.slick.lifted.ProvenShape

case class User(id: Option[Int] = None, name: String)

class UserTable(tag: Tag) extends Table[User](tag, "user") {
  def id: Column[Option[Int]] = column[Option[Int]]("id", O.PrimaryKey, O.AutoInc)

  def name: Column[String] = column[String]("name", O.NotNull)

  override def * : ProvenShape[User] = (id, name) <>(User.tupled, User.unapply)
}
{% endhighlight %}

I'll let to explain about Slick in another post, maybe it'll be better.

Now, let's create a simple object to represent our repository which receives a session implicitly:
{% highlight scala%}
package repositories

import models.{User, UserTable}

import scala.slick.driver.MySQLDriver.simple._
import scala.slick.lifted.TableQuery

object UserRepository {
  private val userTable = TableQuery[UserTable]

  def insert(user: User)(implicit s: Session): Unit = userTable += user
}
{% endhighlight %}

And now let's create our service. Again I am making it simple as a object:
{% highlight scala%}
package services

import models.{User, UserTable}

import scala.slick.driver.MySQLDriver.simple._
import scala.slick.lifted.TableQuery

object UserRepository {
  private val userTable = TableQuery[UserTable]

  def insert(user: User)(implicit s: Session): Unit = userTable += user
}
{% endhighlight %}