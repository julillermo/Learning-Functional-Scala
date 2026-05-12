# Applicative and traversalbe functors

- `applicative functors` are more common, but less powerful, than `monads`
- Will cover how to discover abstractions.
  - Will eventually lead to the discussion on `traversable functors`

## Generalizing monads

- Take note of the following:

```scala
def sequence[A](lfa: List[F[A]]): F[List[A]]
  traverse(lfa)(fa => fa)

def traverse[A,B](as: List[A])(f: A => F[B]): F[List[B]]
  as.foldRight(unit(List[B]()))((a, mbs) => map2(f(a), mbs)(_ :: _))

def map2[A,B,C](ma: F[A], mb: F[B])(f: (A,B) => C): F[C] =
  flatMap(ma)(a => map(mb)(b => f(a,b)))
```

- _"... a large number of the useful combinators on `Monad` can be defined using only `unit` and `map2`."_ (ex. `traverse` combinator)
  - The idea here is that the `Monad` inteface builds on `flatMap` and `unit` primitives. Given this, `Monad` can be represented differently (as with math equations) where `map2` is used instead of `flatMap`, still with `unit`
  - This new "representation" / "abstract" is called the **applicative functor**

## The Applicative trait

- The `Applicative` interfance is on that builds on `map2` and `unit` as it's primitives.
  - All `applicatives` are `functors`.
  - All `monads` are `applicative functors`
    - `Monad[F]` is a subtype of `Applicative[F]`

```scala
trait Appicative[F[_]] extends Functor [F] {
  // primitive combinators
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C]
  def unit[A](a: => A): F[A]

  // derived combinators
  def map[B](fa: F[A])(f: A => B): F[B] =
    map2(fa, unit(()))((a,_) => f(a))

  def traverse[A,B](as: List[A])(f: A => F[B]): F[List[B]]
    as.foldRight(unit(List[B]()))((a, fbs) => map2(f(a), fbs)(_ :: _))
}
```

- Making `Monad` a subtype of `Applicative`

```scala
trait Monad[F[_]] extends Applicative[F] {
  def flatMap[A,B](fa: F[A])(f: A => F[B]): F[B] = join(map(fa)(f))
  def join[A](ffa: F[F[A]]): F[A] = flatMap(ffa)(fa => fa)
  def compose[A,B,C](f: A => F[B], g: B => F[C]): A => F[C] =
    a => flatMap(f(a))(g)
  def map[B](fa: F[A])(f: A => B): F[B] =
    flatMap(fa)((a: A) => unit(f(a)))
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
    flatMap(fa)(a => map(fb)(b => f(a,b)))
}
```

## The difference between monads and applicative functors

- Recall the possible combination of operations that define a `Monad`
  - `unit` and `flatMap`
  - `unit` and `compose`
  - `unit`, `map` andd `join`

### The `Option` applicative versus the `Option` monad

> ... with `Applicative`, the structure of our computation is fixed; with `Monad`, the results of previous computations may influence what computations to run next.

- You might come across functional programmings refer to `Par`, `Option`, `List`, `Parser`, `Gen` as `effects`. This is because these supposedly add abilities. These are sometimes referred to _monadic effects_ or _applicative effects_

### The Parser applicative versus the Parser monad

- skipped

## The advantage of applicative functors

- Using `Monad` would mean that we'd have to implement `traverse`
- It's "weaker" than `Monad`, but comes with more flexibility.
- Applicative functors compose whereas monads don't

### Not all applicative functors are monads

- skipped

## The applicative laws

- Obey functor laws
  - `map(v)(id) == v`
  - `map(map(v)(g))(f) == map(v)(f compose g)`

### 1. Left and Right identity

- ??

### 2. Associativity

- `op(a, op(b, c)) == op(op(a, b), c)`
- `compose(f, op(g, h)) == compose(compose(f, g), h)`

### 3. Naturality of product

- ??

## Traversable Functors

- Recall how we derived applicative functors (generally from Monad ???) by deducing that certain functions can be represented by other primitives.
  - We will do this again for `Traversable Functors`

- Note that there are datatpyes other than list that are `traversable`

- skipped this!

```scala
def traverse[F[_],A,B](as: List[A])(f: A => F[B]): F[List[B]]
def sequence[F[_],A](fas: List[F[A]]): F[List[A]]
```
