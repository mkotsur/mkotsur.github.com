---
layout: post
category : Dev
tags : [scala, java, jvm, scala-java]
---

One more thing to consider when calling Scala function from Java

Let's say we have a few functions that work identical when called from Scala:

```scala
object Hello {

  type Str2Int = String => Int

  def creator: () => Str2Int = () => _.length

  def f1: Str2Int = creator()

  def f2: Str2Int =  (s) => creator()(s)

  def f3(s: String) = creator()(s)

  println(f1("42"))   // 2
  println(f2("422"))  // 3
  println(f3("4222")) // 4
}
```

However, when you try to call them from Java, you'll find out that only `f3` can be applied directly to a `String`. The other 2 will need to be called without arguments, and that will return `Function1<String, Object>`, which you can then call and cast the result to `Int`. Why? Let's look inside into the generated class file with `javap`.

```
javap ./target/scala-2.11/classes/Hello.class
Compiled from "Hello.scala"
public final class Hello {
  public static int f3(java.lang.String);
  public static scala.Function1<java.lang.String, java.lang.Object> f2();
  public static scala.Function1<java.lang.String, java.lang.Object> f1();
  public static scala.Function0<scala.Function1<java.lang.String, java.lang.Object>> creator();
}
```

It's funny that I've run into this issue exactly when the compiler couldn't help: I've generated a function which was supposed to work as a handler for AWS Lambda, and probably it was called using reflection, hence I received no errors at all, the code just hadn't been executed.
