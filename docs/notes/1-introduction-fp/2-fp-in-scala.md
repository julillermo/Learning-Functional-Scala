# Getting started with FP in Scala

- _"In this chapter, we'll begin learning how to write programs in the Scala language just by combining pure functions"_
- _"This chapter is mailny intended for those readers who are new to Scala, to functional programing, or both."_

## Introducing Scala the langauge: an example

- Example code to introducce for familiarity

```scala
// A comment!
/* Another comment */
/** A documentation comment */
object MyModule {
  def abs(n: Int): Int =
    if (n < 0) -n
    else n

  private def formatAbs(x: Int) = {
    val msg = "The absolute value of %d is %d"
    msg.format(x, abs(x)) // The values here are fed to the above message. This is a return value
  }

  // Unit is Scala's equivalent to `void`
  def main(args: Array[String]): Unit =
    println(formatAbs(-42))
}
```

- Scala code has to be in an `object` or a `class`
- When declaring a method, the left-hand side of the equals sign (=) is somethings referred ot as **signature** and the right-hand side as the **definition**
- All expressions produce a result
- The `main` method acts as an impure wrapper to the mostly pure program. Sometimes referred to as `impure functions`.

## Running our program

- You can compile and run specific files using `scalac <path_to_file>` and then run using `scala <path_to_file>` in the CLI
- Howver, Scala projects are typically built using the `sbt` build tool.
- You can also run `scala` and this will trigger a REPL

## Modules, objects, namespaces

- _skipped this one_

## Higher-order functions: passing functions to functions

- Main concept: `functions are values`.
- The term, `Higher-order function (HOF)` refers to accepting other functions as arguments

### Writing loops functionally (recursive functions)

- Loop without the loop syntax
- The `go` function is within the scope so, all of the logic is contained

```scala
def factorial(n: Int): Int = {
  def go(n: Int acc: Int): Int =
    if (n <= 0) acc
    else go (n-1, n*acc)

  go(n, 1)
}
```

- **Tail calls in Scala**
  - _A "tail call" is when a call is in a "tail position". This is when the last line of a logic block calls a function instead of returning value_
  - Note that `go(n,1)` alone is a tail call but `go(n,1)` in `1 + go(n,1)` would not since there's additional work to finish after `go(n,1)` converges on a value.

### Writing our first higher-order function

```scala
object MyModule {
  // ... details of abs() and factorial() here

  private def formatAbs(x: Int) = {
    val msg = "The absolute value of %d is %d."
    msg.format(x, abs(x))
  }
  private def formatFactorial(n: Int) = {
    val msg = "The factorial of %d is %d."
    msg.format(n, factorial(n))
  }

  // has a function as an argument
  def formatResult(name: String, n: Int, f: Int => Int) ={
    val msg = "The %s of %d is %d."
    msg.format(name, n f(n))
  }

  def main(args: Array[String]): Unit = {
    println(formatAbs(-42))
    println(formatFactorial(7))
  }
}
```

## Polymorphic functions

- Code / function that works on any type it's given

```scala
// MONOMORPHIC function to find and element in an array
def findFirstMono(ss: Array[String], key: String): Int = {
  @annotation.talrec
  def loop(n: Int): Int =
    if (n >= ss.length) -1
    else if (ss(n) == key) n
    else loop(n + 1)

  loop(0)
}

// POLYMORPHIC function to find and element in an array
// - It would appear that p() is check condition step
def findFirstPoly(ss: array[A], p: A => Bolean): Int = {
  @annotation.tailrec
  def loop(n: Int): Int =
    if (n >= as.length) -1
    else if (p(as(n))) n
    else (loop (n + 1))

  loop(0)
}
```

### Caslling HOFs with anonymous functions

- Similar to JavaScript, it would be best if we can pass the function as it's declared instead of first having a naming step
- Syntax is also similar to JavaScript (also types are optional)
  - `(x: Int) => x == 9`
  - `(x: Int, y: Int) => x == y`

### Functionas as values in Scala

- Objects that have an `apply` method can be declared such that they work as a function

```scala
val lessThan = new Function2[Int, Int, Boolean] {
  def apply(a: Int, b: Int) = a < b
}
```

## Following types to implementations

- FWIU: you can have an intermediary step called `partial application` where you do some logic but still return a function to be used
  - _"The name comes from the fact that the function is being applied to some but not all of the arguments it requires:"_
    - `def partial1[A,B,C](a: A, f: (A,B) => C): (B) => C`
    - Honestly, I think the above is cleaner than how I recall Haskell was.

# Excercises

- 2.1 recusrive fibonacci
- 2.2 impemented `isSorted` using polymorphism
- 2.3 currying
- 2.4 uncurry
- 2.5 implement HOF that composes 2 functions
