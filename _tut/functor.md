---
layout: default
title:  "Functors"
section: "typeclasses"
source: "https://github.com/non/cats/blob/master/core/src/main/scala/cats/Functor.scala"
scaladoc: "#cats.Functor"
---
# Functor

A Functor is a ubiquitous typeclass involving types that have "one
hole"; that is types which have the shape: `F[?]`, such as `Option`,
`List`, `Future`. (This is in contrast to a type like `Int` which has
no hole, or `Tuple2` which has two "holes" (`Tuple2[?,?]`), etc.

The Functor category involves a single operation, named `map`:

```scala
def map[A, B](fa: F[A])(f: A => B): F[B]
```

This method takes a function from A => B and turns an F[A] into an
F[B].  The name of the method `map` should remind you of the `map`
method that exists on many classes in the Scala standard library. Some
examples of map functions:

```scala
scala> Option(1).map(_ + 1)
res0: Option[Int] = Some(2)

scala> List(1,2,3).map(_ + 1)
res1: List[Int] = List(2, 3, 4)

scala> Vector(1,2,3).map(_.toString)
res2: scala.collection.immutable.Vector[String] = Vector(1, 2, 3)
```

## Creating Functor instances

We can trivially create a functor instance for a type which has a well
  behaved map method:

```scala
scala> import cats._
import cats._

scala> implicit val optionFunctor: Functor[Option] = new Functor[Option] {
     |   def map[A,B](fa: Option[A])(f: A => B) = fa map f
     | }
optionFunctor: cats.Functor[Option] = $anon$1@6f5760df

scala> implicit val listFunctor: Functor[List] = new Functor[List] {
     |   def map[A,B](fa: List[A])(f: A => B) = fa map f
     | }
listFunctor: cats.Functor[List] = $anon$1@493534d2
```

However, functors can also be created for types which don't have a map
method. An example of this would be that Functions which take a String
form a functor using andThen as the map operation:

```scala
scala> implicit def function1Functor[In]: Functor[Function1[In, ?]] =
     |   new Functor[Function1[In, ?]] {
     |     def map[A,B](fa: In => A)(f: A => B): Function1[In,B] = fa andThen f
     |   }
function1Functor: [In]=> cats.Functor[[X_kp1]In => X_kp1]
```

Also of note in the above example, is that we created a functor for
Function1, which is a type which normally has two type holes. We
however constrained one of the holes to be the `In` type, leaving just
one hole for the return type. In this above example, we are
demonstrating the use of the
[kind-projector compiler plugin](https://github.com/non/kind-projector),
This compiler plugin lets us more easily change the number of type
holes a type has. In this case, we took a type which normally has two
type holes, `Function1` and filled one of the holes, leaving the other
hole open. `Function1[In,?]` has the first type parameter filled,
while the second is still open. Without kind-projector, we'd have to
write this as something like: `({type F[A] = Function1[In,A]})#F`,
which is much harder to read and understand.

## Using functor

### map

Option is a functor which always returns a Some with the function
applied when the Option value is a Some.

```scala
scala> val len: String => Int = _.length
len: String => Int = <function1>

scala> Functor[Option].map(Some("adsf"))(len)
res3: Option[Int] = Some(4)

scala> // When the Option is a None, it always returns None
     | Functor[Option].map(None)(len)
res5: Option[Int] = None
```

List is a functor which applies the function to each element the list.
```scala
scala> Functor[List].map(List("qwer", "adsfg"))(len)
res6: List[Int] = List(4, 5)
```

## Derived methods

### lift

We can use the Functor to "lift" a function to operate on the Functor type:

```scala
scala> val lenOption: Option[String] => Option[Int] = Functor[Option].lift(len)
lenOption: Option[String] => Option[Int] = <function1>

scala> lenOption(Some("abcd"))
res7: Option[Int] = Some(4)
```

### fproduct

Functor provides a fproduct function which pairs a value with the
result of applying a function to that value.

```scala
scala> val source = List("a", "aa", "b", "ccccc")
source: List[String] = List(a, aa, b, ccccc)

scala> Functor[List].fproduct(source)(len).toMap
res8: scala.collection.immutable.Map[String,Int] = Map(a -> 1, aa -> 2, b -> 1, ccccc -> 5)
```

## Composition

Functors compose! Given any Functor F[\_] and any Functor G[\_] we can
compose the two Functors to create a new Functor on F[G[\_]]:

```scala
scala> val listOpt = Functor[List] compose Functor[Option]
listOpt: cats.Functor[[X]List[Option[X]]] = cats.Functor$$anon$1@20741e6f

scala> listOpt.map(List(Some(1), None, Some(3)))(_ + 1)
res9: List[Option[Int]] = List(Some(2), None, Some(4))
```
