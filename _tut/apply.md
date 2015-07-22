---
layout: default
title:  "Apply"
section: "typeclasses"
source: "https://github.com/non/cats/blob/master/core/src/main/scala/cats/Apply.scala"
scaladoc: "#cats.Apply"
---
# Apply

Apply extends the Functor typeclass (which features the familiar "map"
function) with a new function "ap".  The ap function is similar to map
in that we are transforming a value in a context, e.g. F[A] where F is
the context (e.g. Option, List, Future) and A is the type of the
value.  But the function A => B is now in the context itself,
e.g. F[A => B] such as Option[A => B] or List[A => B].

```scala
scala> import cats._
import cats._

scala> val intToString: Int => String = _.toString
intToString: Int => String = <function1>

scala> val double: Int => Int = _ * 2
double: Int => Int = <function1>

scala> val addTwo: Int => Int = _ + 2
addTwo: Int => Int = <function1>

scala> implicit val optionApply: Apply[Option] = new Apply[Option] {
     |   def ap[A, B](fa: Option[A])(f: Option[A => B]): Option[B] =
     |     fa.flatMap (a => f.map (ff => ff(a)))
     | 
     |   def map[A,B](fa: Option[A])(f: A => B) = fa map f
     | }
optionApply: cats.Apply[Option] = $anon$1@71691119

scala> implicit val listApply: Apply[List] = new Apply[List] {
     |   def ap[A, B](fa: List[A])(f: List[A => B]): List[B] =
     |     fa.flatMap (a => f.map (ff => ff(a)))
     | 
     |   def map[A,B](fa: List[A])(f: A => B) = fa map f
     | }
listApply: cats.Apply[List] = $anon$1@459b3cc6
```

### map

Since Apply extends Functor, as we expect, we can use the map method
from Functor:

```scala
scala> Apply[Option].map(Some(1))(intToString)
res0: Option[String] = Some(1)

scala> Apply[Option].map(Some(1))(double)
res1: Option[Int] = Some(2)

scala> Apply[Option].map(None)(double)
res2: Option[Int] = None
```


### apply
But also the new apply method, which applies functions from the functor

```scala
scala> Apply[Option].ap(Some(1))(Some(intToString))
res3: Option[String] = Some(1)

scala> Apply[Option].ap(Some(1))(Some(double))
res4: Option[Int] = Some(2)

scala> Apply[Option].ap(None)(Some(double))
res5: Option[Int] = None

scala> Apply[Option].ap(Some(1))(None)
res6: Option[Nothing] = None

scala> Apply[Option].ap(None)(None)
res7: Option[Nothing] = None
```

### ap3, etc

Apply's ap function made it possible to build useful functions that
"lift" a function that takes multiple arguments into a context.

For example:

```scala
scala> val add2 = (a: Int, b: Int) => a + b
add2: (Int, Int) => Int = <function2>

scala> Apply[Option].ap2(Some(1), Some(2))(Some(add2))
res8: Option[Int] = Some(3)
```

Interestingly, if any of the arguments of this example are None, the
final result is None.  The effects of the context we are operating on
are carried through the entire computation.

```scala
scala> Apply[Option].ap2(Some(1), None)(Some(add2))
res9: Option[Int] = None

scala> Apply[Option].ap2(Some(1), Some(2))(None)
res10: Option[Nothing] = None
```

## apply builder syntax

The `|@|` operator offers an alternative syntax for the higher-arity `Apply` functions (`apN`, `mapN`).
First, import `cats.syntax.all._` or `cats.syntax.apply._`. Here we see that following two functions, `f1` and `f2`, are equivalent:

```scala
scala> import cats.syntax.apply._
import cats.syntax.apply._

scala> def f1(a: Option[Int], b: Option[Int], c: Option[Int]) =
     |   (a |@| b |@| c) map { _ * _ * _ }
f1: (a: Option[Int], b: Option[Int], c: Option[Int])Option[Int]

scala> def f2(a: Option[Int], b: Option[Int], c: Option[Int]) =
     |   Apply[Option].map3(a, b, c)(_ * _ * _)
f2: (a: Option[Int], b: Option[Int], c: Option[Int])Option[Int]

scala> f1(Some(1), Some(2), Some(3))
res11: Option[Int] = Some(6)

scala> f2(Some(1), Some(2), Some(3))
res12: Option[Int] = Some(6)
```

All instances created by `|@|` have `map`, `ap`, and `tupled` methods of the appropriate arity.

## composition

Like Functors, Apply instances also compose:

```scala
scala> val listOpt = Apply[List] compose Apply[Option]
listOpt: cats.Apply[[X]List[Option[X]]] = cats.Apply$$anon$1@7102beb0

scala> val plusOne = (x:Int) => x + 1
plusOne: Int => Int = <function1>

scala> listOpt.ap(List(Some(1), None, Some(3)))(List(Some(plusOne)))
res13: List[Option[Int]] = List(Some(2), None, Some(4))
```
