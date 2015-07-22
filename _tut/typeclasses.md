# Typeclasses

The typeclass pattern is a ubiquitous pattern in Scala, its function
is to provide a behavior for some type. You think of it as an
"interface" in the Java sense. Here's an example.

```scala
scala> /**
     |  * A typeclass to provide textual representation
     |  */
     | trait Show[A] {
     |   def show(f: A): String
     | }
defined trait Show
```
This class says that a value of type `Show[A]` has a way to turn `A`s
into `String`s. Now we can write a function which is polymorphic on
some A, as long as we have some value of Show[A], so that our function
can have a way of producing a String:

```scala
scala> def log[A](a: A)(implicit s: Show[A]) = println(s.show(a))
log: [A](a: A)(implicit s: Show[A])Unit
```

If we now try to call log, without supplying a Show instance, we will
get a compilation error:

```scala
scala> log("a string")
<console>:14: error: could not find implicit value for parameter s: Show[String]
       log("a string")
          ^
```

It is trivial to supply a Show instance for String:

```scala
scala> implicit val stringShow = new Show[String] {
     |   def show(s: String) = s
     | }
stringShow: Show[String] = $anon$1@4125c297

scala> // and now our call to Log succeeds
     | log("a string")
a string
```

This example demonstrates a powerful property of the typeclass
pattern. We have been able to provide an implementation of Show for
String, without needing to change the definition of java.lang.String
to extend a new Java-style interface; something we couldn't have done
even if we wanted to, since we don't control the implementation of
java.lang.String. We use this pattern to retrofit existing
types with new behaviors. This is usually referred to as "ad-hoc
polymorphism".

For some types, providing a Show instance might depend on having some
implicit Show instance of some other type, for instance, we could
implement Show for Option:

```scala
scala> implicit def optionShow[A](implicit sa: Show[A]) = new Show[Option[A]] {
     |   def show(oa: Option[A]): String = oa match {
     |     case None => "None"
     |     case Some(a) => "Some("+ sa.show(a) + ")"
     |   }
     | }
optionShow: [A](implicit sa: Show[A])Show[Option[A]]
```

Now we can call our log function with a `Option[String]` or a
`Option[Option[String]]`:

```scala
scala> log(Option(Option("hello")))
Some(Some(hello))
```

Scala has syntax just for this pattern that we use frequently:

```scala
def log[A : Show](a: A) = println(implicitly[Show[A]].show(a))
```

is the same as

```scala
def log[A](a: A)(implicit s: Show[A]) = println(s.show(a))
```

That is that declaring the type parameter as `A : Show`, it will add
an implicit parameter to the method signature (with a name we do not know).
