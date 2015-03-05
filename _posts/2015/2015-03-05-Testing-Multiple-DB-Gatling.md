---
layout: post
title: Testing Multiple Database using Gatling
---

Hi there, everythin's fine?

After my previous [post]({% post_url 2015/2015-01-24-Multiple-DB %}) I was wondering how I could write tests for it.  
Searching on Google I've found an article explaining how to use [Gatling](http://gatling.io/) to write stress tests.
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

First of all create a new Scala project called __gatling-multidb__.

After that place the following lines in the __plugins.sbt__ file located on project directory:

{% highlight scala %}
addSbtPlugin("io.gatling" % "gatling-sbt" % "2.1.0")
{% endhighlight %}

This is needed to allow sbt run tests.

After place Gatling sbt plugin we need to place Gatling dependecies to our project changing the __build.sbt__ file as following:  

{% highlight scala %}
import io.gatling.sbt.GatlingPlugin

name := """gatling-multidb"""

version := "1.0"

scalaVersion := "2.11.6"

val integration = "latest.integration"

enablePlugins(GatlingPlugin)

libraryDependencies ++= Seq(
  "org.scalatest" %% "scalatest" % integration % "test",
  "io.gatling" % "gatling-core" % integration % "test",
  "io.gatling.highcharts" % "gatling-charts-highcharts" % integration % "test",
  "io.gatling" % "gatling-test-framework" % integration % "test"
)
{% endhighlight %}

*Note:* Do not forget to run __reload__ before __update__ if you already have started activator.

It's time to write some code.

Let's create a class __MultidbSimulation__ called in the __test__ directory:

{% highlight scala %}
import io.gatling.core.Predef._
import io.gatling.http.Predef._

class MultiDBSimulation extends Simulation {

  // the test scenario
  val scn = scenario("MultiDBScenario").repeat(1) { // This scenario are going to be repeated once
    exec(http(session => "Request User 1")
      .post("http://localhost:9000/user/user1") // URL to post using user1 database
      .check(status is 200, substring("Logged: user1")) // Verify if status is 200 and response body contains this this string
    ).exec(http(session => "Request user 2")
      .post("http://localhost:9000/user/user2") // URL to post using user2 database
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

This class is our test class. That's where we configure the test's scenarios. It's a little bit simple but we can do interesting things
using Gatling. It's nice to read the documentation, I'm doing it.

To test just start our [Multiple DB application]({% post_url 2015/2015-01-24-Multiple-DB %}). After that, in the __gatling-multidb__
directory type __activator__ and after type __test__. When it's finished we can check a very nice chart about our tests just typing
__lastReport__ on terminal.


You can check out this source code [here](https://github.com/gbmetzner/test-multipledb)

That's all folks!

I hope I could help.

If you have questions let me know.
