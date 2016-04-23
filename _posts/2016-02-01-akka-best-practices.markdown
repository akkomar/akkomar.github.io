---
published: true
title: Akka - best practices
layout: post
tags: [akka]
---
# Creating actors
Instead of creating actors by type signature:

```scala
val actor = context.actorOf(Props[ApiActor])
```

Do this:

```scala
val actor = context.actorOf(Props(new ApiActor()))
```
This way if you later add constructor arguments to `ApiActor`, you'll immediately get compile error
(which is not the case if actor was created by type signature).


The important thing we have to remember if creating actors this way is that we should not use it from within
another actor as it allows to close over an enclosing actor. It can result in non-serializable Props,
and more surprisingly it makes it possible to get a handle to "parent" actor via reflection
(because JVM stores information about "parent" object).

Instead, we can create Props factory in companion object of our actor:

```scala
object ApiActor {
  def props(apiUrl: String): Props = Props(new ApiActor(apiUrl))

  case class CallApi(msg: String)
}
```

Above snippet shows another nice trick which makes your code easier to understand -
 we can define all messages our actor can receive in its companion object.

