# Intro

- If we can't mutate variables, what kinds of data can we work on with Scala?
- Will cover:
  - functional data structures
  - pattern matching

## Defining functional data structures

- DEFINITION: data structures only operated on by pure functions (are immutable)
- Aren't we greatly increasing the workload of copying / recreating data?
  - No, to be explained later
- Specifying `A` indicates that the datatype is `polymorphic`. The `A` acts like a placeholder
  - `List[String]`, `List[Double]`, etc.
- **variance**
  - The addition of `+` in front of `A` in `+A` indicates that `List[+A]` is a **covariant** or "pisitive" parameter of list
  - If `Dog` is a subtype of `Animal`, then `List[Dog]` is a subtype of `List[Animal]`

```scala
// SIngly linked lists
package fpinscala.datastructures

sealed trait List[+A]
case object Nil extends List[Nothing]
case class Cons[+A](head: A, tail: List[A]) extends List[A]

object List {
  def sum(ints: List[Int]): Int = ints match {
    case Nil => 0
    case Cons(x,xs) => x + sum(xs)
  }

  def product(ds: List[Double]): Double = ds match {
    case Nil => 1.0
    case Cons(0.0, _) => 0.0
    case Cons(x,xs) => x * product(xs)
  }

  def apply[A](as: A*): List[A]
    if (as.isEmpty) Nil
    else Cons(as.head, apply(as.tail: _*))
}
```

## Patttern matching

- Still referring to the previous code block
- FWIU: this is also covered by my other scala reference, specificaly on match-case syntax / pattern matching
- Works like a `switch` statement (I assume the reference is Java)
- Some examples
  - `List(1,2,3) match { case _ => 42 }` results in `42`
  - `List(1,2,3) match { case Cons(h,_) => h }` results in `1`
  - `List(1,2,3) match { case Cons(_,t) => t }` result `List(2,3)`
  - `List(1,2,3) match { case Nil => 42 }` results in a `MatchError`

### Companion objects in Scala

> We'll often declare a _companion object_ in addition to our data type and its data constructors. This is just an `object` with the same as the data type (in this case `List`) where we put various convenience functions for creating or working with values of the data type.

## Data Sharing in functional data structures

- Since data is immutable, we return a new list when we want to modify it.
- The lists are reused in a process referred to as `data sharing`
  - I think this is the answer to the hypothetical question at the start. Data is shared, so even when the variable is immutable, there's no major cost to "returning a new list"
  - ex. removing the first value is simply using `tail` of that "string" of values. _"There's no real removing oing on"_

### Appending an entire list onto another

- Cons can also be represented as `h :: t`

```scala
def append[A](a1: List[A], a2: List[A]): List[A] =
  a1 match {
    case Nil => a2
    case Cons(h,t) => Cons(h append(t, a2))
  }
```

### Improving type inference for HOFs

- Implement custom `dropWhile`

```scala

def dropWhileOrig[A](l: List[A], f: A => Boolean): List[A]

def dropWhileCust[A](as: List[A])(f: A => Boolean): List[A] =
  as match {
    case Cons(h,t) if f(h) => dropWhile(t)(f)
    case _ => as
  }

val xs: List[Int] = List(1,2,3,4,5)
val ex1 = dropWhileCust(xs)(x => x < 4)
```

## Recursion over lists and generalizing to higher-order functions

## Recursion over lists and generalizing to HOFs

- Still referring to the same implementation of `sum` and `product`

```scala
def sum(ints: List[Int]): Int = ints match {
  case Nil => 0
  case Cons(x,xs) => x + sum(xs)
}

def product(ds: List[Double]): Double = ds match {
  case Nil => 1.0
  case Cons(x, xs) => x * product(xs)
}
```

- Using right folds
  - Not that `.foldRight` and `.foldLeft` are built-in methods to lists in Scala

```scala
def foldRightCust[A,B](as: List[A], z: B)(f: (A,B) => B): B =
  as match {
    case Nil => z
    case Cons(x,xs) => f(x, foldRightCust(xs, z)(f))
  }

def sum2(ns: List[Int]) =
  foldRight(ns,0)((x,y) => x + y)

def product2(ns: List[Double]) =
  foldRight(ns, 1.0)(_ * _)
```

## Trees

- It tried to cover `LEFT` and `RIGHT` as trees, but was hard to interpret

## Excercises

- 3.1
- 3.2 implement the tail for removing the first element of a `List`
- 3.3 implement `setHead`
- 3.4
- 3.5 implement `dropWhile`
- 3.6
- 3.7
- 3.8
- 3.9
- 3.10
- 3.11
- 3.12
- 3.13
- 3.14
- 3.15
  ...
