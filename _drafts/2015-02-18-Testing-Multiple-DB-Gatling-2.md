---
layout: post
title: Testing Multiple Database using Gatling 2
---

Hi there, everythin's fine?

After my previous [post]({% post_url 2015/2015-01-24-Multiple-DB %}) I was wondering how I could write tests for it.  
Searching on Google I've found an article explaining how to use [Gatling 2](http://gatling.io/) to write stress tests.
You can check out this article [here](http://sysgears.com/articles/restful-service-load-testing-using-gatling-2/)

According to the Gatling website:  

> Gatling is a highly capable load testing tool. It is designed for ease of use, maintainability and high performance.
> 
> Out of the box, Gatling comes with excellent support of the HTTP protocol that makes it a tool of choice for load testing any HTTP server. As the core engine is actually protocol agnostic, it is perfectly possible to implement support for other protocols. For example,Gatling currently also ships JMS support.
> 
> Having scenarios that are defined in code and are resource efficient are the two requirements that motivated us to create Gatling. Based on an expressive DSL, the scenarios are self explanatory. They are easy to maintain and can be kept in a version control system.
> 
> Gatling’s architecture is asynchronous as long as the underlying protocol, such as HTTP, can be implemented in a non blocking way. This kind of architecture lets us implement virtual users as messages instead of dedicated threads, making them very resource cheap. Thus, running thousands of concurrent virtual users is not an issue.

Thus, in this post I'm going to show you how I've written the stress test for our [Multiple DB application]({% post_url 2015/2015-01-24-Multiple-DB %}).

First of all create a new Scala project called --gatling-multidb--.

After that place the following lines in the --plugins.sbt-- file located on projects directory:

{% highlight scala %}
addSbtPlugin("io.gatling" % "gatling-sbt" % "2.1.0")
{% endhighlight %}

This is needed to allow sbt run test on the terminal.

After place Gatling sbt plugin we need to place Gatling dependecies to our project changing the build.sbt file as following:  

{% highlight scala %}
import io.gatling.sbt.GatlingPlugin

name := """gatling-multidb"""

version := "1.0"

scalaVersion := "2.11.5"

val integration = "latest.integration"

enablePlugins(GatlingPlugin)

libraryDependencies ++= Seq(
  "org.scalatest" %% "scalatest" % "2.2.4" % "test",
  "io.gatling" % "gatling-core" % integration % "test",
  "io.gatling.highcharts" % "gatling-charts-highcharts" % integration % "test",
  "io.gatling" % "gatling-test-framework" % integration % "test"
)
{% endhighlight %}

*Note:* Do not forget to run --reload-- before --update-- if you already have started activator.

It's time to write some code.

Let's create a class --MultidbSimulation-- called in the --test-- directory:

{% highlight scala %}
import io.gatling.core.Predef._
import io.gatling.http.Predef._

class MultiDBSimulation extends Simulation {

  // the test scenario
  val scn = scenario("MultiDBScenario").repeat(1) {
    exec(http(session => "Request User 1")
      .post("http://localhost:9000/user/user1")
      .check(status is 200, substring("Logged: user1"))
    ).exec(http(session => "Request user 2")
      .post("http://localhost:9000/user/user2")
      .check(status is 200, substring("Logged: user2"))
      )
  }

  // application url
  val httpProtocol = http.baseURL("http://localhost:9000")

  // how many users
  val oneThousand = 1000

  setUp(scn.inject(atOnceUsers(oneThousand))).protocols(httpProtocol)
    .assertions(global.successfulRequests.percent.is(100))

}
{% endhighlight %}

To test just start our [Multiple DB application]({% post_url 2015/2015-01-24-Multiple-DB %}) and run --test-- on activator console.

