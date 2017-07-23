---
layout: post
category : Dev
tags : [scala, type variance, humor]
title: T-shirt [T]
---

Had a nice chat with a colleague after returning to work from ScalaSwarm conference:

- Me: I brought a cool T-shirt
- He: Is it a Scala T-shirt?
- No, but it's still cool. T-shirt is an invariant type with regards to coolness.

![Output in terminal](/static/img/2017-07-23-t-shirt-of-t/tshirt.png)

That seemed like a good joke at that moment, but later I realized that variance is applicable only to subtyping relationships, and not to coolness. What I actually meant was that if there are cool and normal things:

```scala
sealed trait Cool
object Scala extends Cool // cool
object CommunistParty // normal
```

Than this is perfectly possible:

```scala
class `Cool_T-Shirt`[T]
val tShirt = new `Cool_T-Shirt`[CommunistParty.type]
```

It would *not* be possible of there was an upper type bound on type parameter of `Cool_T-Shirt` type, that would allow only _Cool_ things to be passed into.

```scala
class `Cool_T-Shirt`[T <: Cool]
val tShirt = new `Cool_T-Shirt`[CommunistParty.type]

// Error:(9, -69) type arguments [A$A64.this.CommunistParty.type] do not conform to class Cool_T-Shirt's type parameter bounds [T <: A$A64.this.Cool]
// lazy val tShirt = new `Cool_T-Shirt`[CommunistParty.type];}
//    ^
```

In normal words that would mean: T-Shirts can be cool only if the thing depicted on it is cool, which is not true, in my opinion. To make a good joke about this I should have said:

- No, but it's still cool. `Cool_T-shirt` doesn't have bounds on type of the image.
