---
layout: post
category : Dev
tags : [spark, databricks, scala, sbt]
title: Writing in IntelliJ, running on Databricks
---

So, Apache Spark is getting popular and I'm starting to get a hang of it. Our team uses [Databricks](https://databricks.com/), and I found it (and the whole concept of notebooks) great for small things. However, when it comes to writing more complex programs, I'm very much used to convenience of IntelliJ for writing, testing and refactoring the code. Moreover, the new [Dataset[T]](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/Dataset.html) API allows you to rely on type safety and depend much less on constant re-runs.

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

## Step 2: Write

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

The last line will contain an unresolved variable, but the rest of the code will be treated as valid.

## Step 3: Run

Hit `dbcExecuteCommand` in sbt console, and you should see something like like this:
![Output in terminal](/static/img/2017-03-19-running-databricks-from-intellij/output.png)
