---
layout: post
category : Dev
tags : [spark, databricks, scala, sbt]
title: Running Databricks from IntelliJ
---

So, Apache Spark is getting popular and I'm starting to get a hang of it. Our team uses [Databricks](https://databricks.com/), and I found it (and the whole concept of notebooks) great for sharing and visually presenting programs that deal with data. However, as a developer, I'm very much used to convenience of IntelliJ for writing, testing and refactoring the code.

In my opinion, lack of 'native' support for autocompletion, error indication, simple refactoring, and other features that are considered a 'must have' in modern IDEs leads to slower learning, longer write-run-fix cycle and sloppier code. This post is intended to show how to have all these features while running jobs with Databricks.

## Step 1: Connect with Databricks

Create a simple SBT project and [add sbt-databricks plugin](https://github.com/databricks/sbt-databricks#installation) to the project, [access details should be specified](https://github.com/databricks/sbt-databricks#settings) in `build.sbt`.

In `build.sbt` you should have somthing like:

```sbt
name := "databricks-test"
version := "1.0"
scalaVersion := "2.11.8"

libraryDependencies += "org.apache.spark" %% "spark-core" % "2.0.0"
libraryDependencies += "org.apache.spark" %% "spark-sql" % "2.0.0"

dbcUsername := "myself@example.com" // Databricks username
dbcPassword := "verysecret" // and password

// The URL to the Databricks Cloud API!
dbcApiUrl := "https://community.cloud.databricks.com/api/1.2"

dbcClusters += "My Cluster" // Where your stuff should be run

dbcExecutionLanguage := DBCScala // Language

dbcCommandFile := baseDirectory.value / "src/main/scala/mean.scala"
```

the last settings points to a Scala script. Create it and let's start coding.

## Step 2: Implement your job

In contrast to notebooks, all your code will end up in one chunk, which may or may not be what you want, but as a developer you know a number of ways of stopping the execution where you need.

There are several things like `sc: SparkContext` that are globally available in Databricks. IntelliJ doesn't have a clue about those, so to have a nice autocompletion we need to do something about it. I've found this pattern pretty handy:

```scala
import org.apache.spark.SparkContext
import org.apache.spark.sql._

def run(sc: SparkContext, ss: SparkSession) = {
  import ss.implicits._

  val numbers: Dataset[Int] = Seq(1, 2, 3).toDS()
  val doubled: Dataset[Int] = numbers.map(_ * 2)

  println("=== The new dataset is: ====")
  doubled.show()

  println("=== Mean of the new dataset is: ====")
  println(doubled.reduce(_ + _) / doubled.count())
}

run(sc, SparkSession.builder().getOrCreate())
```

IntelliJ will report the error in the last line, but the rest of the code will be valid.

## Step 3: run

Hit `dbcExecuteCommand` in sbt console, and you should see something like like this:
![Output in terminal](/static/img/2017-03-19-running-databricks-from-intellij/output.png)
