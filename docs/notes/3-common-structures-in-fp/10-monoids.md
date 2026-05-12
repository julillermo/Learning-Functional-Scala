# Monoids

- Introduction to **purely algebraic** structures
- The **monoid** is defined **only by its algebra**
- Demonstrate:
  - monoids allow us to chunk the problem which opens the possibility of parallel computation
  - the chunks are composable and be assembled to solve complex problems

## What is a monoid?

- Had a discussion on properties of ceratin basic operations. Addition & multiplication is associative and so are code logic like string concatenation (same order of text) or the boolean operators `&&` and `||`.
  - This algebra is called as **monoid**
  - The laws that govern them (ex. identity law, etc.) are known as **monoid laws**
- monoids consist of the following:
  - Some type `A`
  - An associative binary operation, `op` that takes 2 values of type A and combines them into 1
    - `op(op(x,y),z) == op(x,op(y,z))`
  - When `zero: A`, then `op(x,zero) == x` & `op(zero,x) == x` given `x: A`
- The following is the scala representation of a monoid

```scala
trait Monoid[A] {
  def op(a1: A, a2: A): A
  def zero: A
}

val stringMonoid = new Monoid[String] {
  def op(a1: String, a2: String) = a1 + a2
  val zero = ""
}

def listMonoid[A] = new Monoid[List[A]] {
  def op(a1: List[A], a2: List[A]) = a1 ++ a2
  val zero = Nil
}
```

- Answer:
  - The monoid is a **type** and **set of laws**, it's the algebra
  - It's more accurate to say that `stringMonoid` and `listMonoid` **form** a monoids rather than being monoids

> a monoid is a type together with a binary operation (op) over that type, satisfying associativity and having an identit element (zero)

## Folding lists with monoids

- If you're using `foldLeft` and `foldRight` on a `List` (containing elements with the same type), this satisfies conditions for a monoid:
  - In this scenario, I think the List with the same type `List[A]` are the monoids?

```scala
//general type definition
def foldRight[B](z:B)(f: (A,B) => B): B
def foldLeft[B](z:B)(f:(B,A) => B): B

// When same type (A == B)
def foldRight[A](z:A)(f: (A,A) => A): A
def foldLeft[A](z:A)(f:(A,A) => A): A
```

## Associativity and Parallelism

- The **associativity** aspect of monoids allows us to choose how the operation is applied
  - Ex. fold direction in `foldRight` and `foldLeft`
  - `foldRight` -> `op(a, op(b, op(c, d)))`
  - `foldLeft` -> `op(op(op(a,b),c),d)`
- We should also be able to have a **balanced fold**:
  - `op(op(a,b), op(c,d))`
  - Notice the parellelism the above statement lends to
  - Again, I think this is only specific to folding with monoids because the monoid properties:
    - same type
    - op() operation is associative

## Parallel Parsing

- Consider that we wanted to split a really long string to count the number of words
  - Sequential logic works, but would can be longer than running something in parallel
- As with the above, this parallel word count should be doable under the monoid consideration
- Attempt to parse `"lorem ipsum dolor sit amet"` as small example

```scala
sealed trait WC
case class Stub(chars: String) extends WC
case class Part(lStub: String, words: Int, rStub: String) extends WC
// lStub contains any partial word to the left of the words,
// rStub contains any partial word to the right of the words
```

- This part ended without making sense to me

## Foldable data structures

```scala
trait Foldable[F[_]] {
  def foldRight[A,B](as: F[A])(z: B)(f: (A,B) => B): B
  def foldLeft[A,B](as: F[A])(z: B)(f:(B,A) => B): B
  def foldMap[A,B](as: F[A])(f: A => B)(mb: Monoid[B]): B
  def concatenate[A](as: F[A])(m: Monoid[A]): A = FoldLeft(as)(m.zero)(m.op)
}
```

- In the above `F[_]` is a type constructor that takes 1 argument.
  - `Foldable` is a `higher-order type constructor`

## Composing monoids

- The real power of `monoids` is that they can be composed.
  - If types `A` and `B` are monoids, then their product, tuple `(A, B)`, is also a monoid

### More complex monoids

```scala
// Merging key-value maps
def mapMergeMonoid[K,V](V: Monoid[V]): Monoid[Map[K,V]] =
  new Monoid[Map[K,V]] {
    def zero = Map[K,V]()
    def op(a: Map[K,V], b: Map[K,V]) =
      (a.keySet ++ b.keySet).foldLeft(zero) {
        (acc,k) => acc.updated(k,V.op(a.getOrElse(k,V.zero),b.getOrElse(k,V.zero)))
      }
  }
```

### Using composed monoids to fuse traversals

> The fact that multiple monoids can be composed into one means that we can perform multiple calculations simultaneously when folding a data structure.
