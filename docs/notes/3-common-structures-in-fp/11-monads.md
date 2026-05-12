# Monads

- _"... we'll continue this mode of thinking and apply it to the problem of factoring out code ducpliation across the libraries we wrote in parts 1 and 2"_
- New abstract interfaces to discuss
  - `Functor` and `Monad`

## Functors: generalizing the `map` function

- The following class/type specific `map` functions were derived in the previous chapters

```scala
def map[A,B](ga: Gen[A])(f: A => B): Gen[B]
def map[A,B](ga: Parser[A])(f: A => B): Parser[B]
def map[A,B](ga: Option[A])(f: A => B): Option[B]
```

- The above share the general code pattern, and the type signature can be generalized via the following Scala trait
- FWIU: the `Functor` can be used as a generalization for tpyes like `Gen`, `Parser`, and `Option`

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
  - _"This kind of algebraic reasoning can potentially save us a lot of work, since we donot; ahve to write separate tests for these properties."_

## Monads: generalizing the `flatMap` and unit functions

- _"`Functor` is just one of many abstractions that we can factor out of our libraries."_
- _"Using this interface, we can implement a number of useful operations, once and for all factoring out what would otherwise be duplicated code."_ It comes with laws that we always assume as upheld (similar to `monoid`).

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

- It's quite noticeable how identical the above implementations are.
- We should be able to generalize `map2` so it's not specific to a type.

### The Monad trait

- The commonality of `Parser`, `Gen`, `Par`, `Option` is that they're `monads`. We should be able to generalize it using a `trait`. We can do this like we did in the above

```scala
// Mon trait with map2
trait Mon[F[_]] {
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
    fa flatMap (a => fb map (b => f(a,b)))
}
```

- extending/specifying the `Mon` trait to include `map` and `flatMap`. Otherwise, this won't compile (in the assumption that `flatMap` and `map` are unavailable)

```scala
trait Mon[F[_]] {
  def map[A,B](fa: F[A])(f: A => B): F[B]
  def flatMap[A,B](fa: F[A])(f: A => F[B]): F[B]

  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
    fa flatMap (a => fb map (b => f(a,b)))
}
```

- The above builds the `map2` function using `map` and `flatMap` as primitives. However map itself can be represented via `unit` and `flatMap`.

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

- A lot of these were execises on implementing the other methods that should be available in the built-in implmenetation of Monad

## Monad laws

- The functor laws should hold for `Monad` since `Monad[F]` _is_ a `Functor[F]`

### The associative law

- Refer to [[10-monoids]] for the associative law, and why this is being mentioned
  - `compose(compose(f,g),h) == compose(f, compose(g,h))`

### The identit laws

-

## Just what is a Monad?

> Yes, `Monad` factors out code duplication among them [previous examples], but what _is_ a monad exactly?

- Similar to `Monoid`, a `Monad` is a _purely algebraic interface_
- ... instances of `Monad` will have to provide implementations of one of these sets:
  - `unit` and `flatMap`
  - `unit` and `compose`
  - `unit`, `map` andd `join`

> A monad is an implementation of one of the minimal sets of monadic combinators [I think referring to the above], satisfying the laws of associativity and identity. [the only correct definition]

### The identity monad

- Give the following, the succeeding REPL code turns out as such

```scala
case class Id[A](value: A) {
  def map[B](f: A => B): Id[B] =
    Id(f(value))

  def flatMap[B](f: A => Id[B]): Id[B] =
    f(value)
}
```

```scala
scala> Id("Hello, ") flatMap (a => Id("monad!") flatMap (b => Id(a + b)))
val res1: Id[String] = Id(Hello, monad!)

scala> for {
     |  a <- Id("Hello, ")
     |  b <- Id("monad!")
     | } yield a + b
val res2: Id[String] = Id(Hello, monad!)
```

- We could have just accomplished the same without the **Id** wrapper

```scala
val a = "Hello, "
val b = "monad!"

val combined = a + b // "Hello, monad!"
```

> Besides the `Id` wrapper, there's no difference. So now we have at least a partial answer to the question of what monads mean. We could say that monads provide a context for introducing and binding variables, and performing variable substitution

### The State monad and partial type application

- Previous `State` data type example appears to satisfy our requirements for being a monad

```scala
case class State[S, A](run: S => (A, S)) {
  def map[B](f: A => B): State[F, B] =
    State(s => {
      val (a, s1) = run(s)
      (f(a) s1)
    })
  def flatMap[B](f: A => State[S, B]): State[S, B] =
    State(s => {
      val (a, s1) = run(s)
      f(a).run(s1)
    })
}
```

- It fails in that a construtor expects only 1 argument, so `Monad[State]` is invalid. This is remedied by the idea that **state has a whole family of monads for each choice of `S`** `State[S,_]`
- The above is supposedly like the `IntState[A]` type where State takes 2 arguments, but `IntState[A]` only takes 1
  - `type IntState[A] = State[Int, A]`
- `IntState[A]` exactl aligns with out requirement for `Monad`

```scala
def stateMonad[S] = new Monad[({type f[x] = State[S,x]})#f] {
  def unit[A](a: => A): State[S,A] = State(s => (a,s))
  def flatMap[A,B](st: State[S,A])(f: A => State[S,B]): State[S,B] =
    st flatMap f
}
```

## Exercises

- 11.3 Includding the `sequence` and `traverse` methods to monads
