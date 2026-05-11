# Monads

- _"... we'll continue this mode of thinking and apply it to the problem of factoring out code ducpliation across the libraries we wrote in parts 1 and 2"_
- Abstract interfaces
  - `Functor` and `Monad`

## Functors: generalizing the `map` function

- The following class/type specific `map` functions were derived in the previous chapters

```scala
def map[A,B](ga: Gen[A])(f: A => B): Gen[B]
def map[A,B](ga: Parser[A])(f: A => B): Parser[B]
def map[A,B](ga: Option[A])(f: A => B): Option[B]
```

- The above share the general code pattern, and the type signature can be generalized via the following Scala trait

```scala
trait Functor[F[_]] {
  def map[A,B](fa: F[A])(f: A => B): F[B]

  // This is essentially the same as unzip for any functor
  def distribute[A,B](fab: F[(A,B)]): (F[A], F[B]) =
    (map(fab)(_._1), map(fab)(_._2))

  def codistribute[A,B](e: Either[F[A], F[B]]): F[Either[A, B]] =
    e match {
      case Left(fa) => map(fa)(Left(_))
      case Right(fa) => map(fb)(right(_))
    }
}

val listFunctor = new Functor[List] {
  def map[A,B](as: List[A])(f: A => B): List[B] = as map f
}
```

### Functor laws

- Laws are important for 2 reasons
  - _"Laws help an interface form a new semantic level whose algebra may be reasoned about `independently` of the instances"_
- There was a discussion on the identity function
  - _"Since we know nothing about `F` other than it's a functor, the law assures us that the returned values will have the same shape as the arguments"_

## Monads: generalizing the `flatMap` and unit functions

- _"Using this interface, we can implement a number of useful operations, once and for all factoring out what would otherwise be duplicated code."_ It comes with laws that we always assume as upheld.

```scala
// Implementing map2 for Gen, Parser, and Option
def map2[A,B,C](fa: Gen[A], fb: Gen[B])(f: (A,B) => C): Gen[C] =
  fa flatMap (a => fb map (b => f(a,b)))
```

```scala
def map2[A,B,C](fa: Parser[A], fb: Parser[B])(f: (A,B) => C): Parser[C] =
  fa flatMap (a => fb map (b => f(a,b)))
```

```scala
def map2[A,B,C](fa: Option[A], fb: Option[B])(f: (A,B) => C): Option[C] =
  fa flatMap (a => fb map (b => f(a,b)))
```

- We should be able to generalize `map2` so it's not specific to a type

### The Monad trait

- The commonality of `Parser`, `Gen`, `Par`, `Option` is that they're `monads`. We should be able to generalize it using a `trait`. We can do this like we did in the above

```scala
// Mon trait with map2
trait Mon[F[_]] {
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
    fa flatMap (a => fb map (b => f(a,b)))
}
```

- extending/specifying the `Mon` trait to include `map` and `flatMap`

```scala
trait Mon[F[_]] {
  def map[A,B](fa: F[A])(f: A => B): F[B]
  def flatMap[A,B](fa: F[A])(f: A => F[B]): F[B]

  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
    fa flatMap (a => fb map (b => f(a,b)))
}
```

- The above builds the `map2` function using `map` and `flatMap` as primitives. However map itself can be represented via `unit` and `flatMap`

```scala
trait Mon[F[_]] {
  // Unit is the function that puts a plain value into a monadic context
  def unit[A](a: => A): F[A]
  def flatMap[A,B](fa: F[A])(f: A => F[B]): F[B]

  def map[A,B](ma: F[A])(f: A => B): F[B] =
    flatMap(ma)(a => unit(f(a)))
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
    fa flatMap (a => fb map (b => f(a,b)))
}
```

```scala
// Implementing `Monad` for `Gen`
object Monad {
  val genMonad = new Monad[Gen] {
    def unit[A](a: => A): Gen[A] = Gen.unit(a)
    def flatMap[A,B](ma: Gen[A])(f: A => Gen[B]): Gen[B] =
      ma flatMap f
  }
}
```

## Monadic combinators

- A lot of these where execises in implementing the other methods that should be available in the built-in implmenetation of Monad

## Monad laws

- The functor laws should hold for `Monad` since `Monad[F]` _is_ a `Functor[F]`

### The associative law

-

## Exercises

- 11.3 Includding the `sequence` and `traverse` methods to monads
