# Handling errors without exceptions

- Throwing exceptions is a side effect
  - _"if exceptions aren't used in functional code, what is used instead?"_
  - _"The functional solution, of returning errors as values, is safer and retains referential transparency, and through the use of higher-order functions, we can preserve the primary benefit of exceptions--consolidation of error-handling logic"_
- This chapter will cover recreation of `Option` and `Either`
  - use the built-in `Option` and `Either` after going through this

## Good and bad aspects of Exceptions

- The following way to handle errors is not **referentially transparent**
  - different input returns the same result
- _"exceptions break RT and introduce context dependence"_
- _"Exceptions are not type-safe"_

```scala
def failingFn(i: Int): Int = {
  val y: Int = throw new Exception("fail!")
  try {
    val x = 42 + 5
    x + y
  } catch {
    case e: Exception => 43
  }
}
```

## Possible alternatives to exceptions

- Remedy: instead of throwing exception, return a value indicating an exception condition occured.
- skimmed

## The Option data type

- It's nice that it's explicitly stating which methods are available to options, but I'm more interested in learning how to use them at the moment.
- The `Option` type is also part of my other Scala reference. FWIU: Option is mainly for describing whether a list is empty or has contents:
  - `Option`
    - `Some`
    - `None`

## The Either data type

- This wasn't covered in the online book
- The `Option/Some/None` has a limitation in that it only returns `None` (no value), but more information is sometimes needed.
  - If we only needed to know an error occured, `Option` is fine.
- You can interpret `Option` as an **extension** `Either`
  - Note that `Either` always leads to a case with a value (values that can be 1 of 2 things)
  - The `Left` constructor is reserved for **failure case** by convention
  - The `Right` constructor is reserved for **success case** by convention

```scala
sealed trait Either[+E, +A]
case class Left[+E](value: E) extends Either [E, Nothing]
case class Right[+A](value: A) extends Either[Nothing, A]
```

```scala
def mean(xd: IndexedSeq[Double]): Either[String, Double] =
  if (xs.isEmpty)
    Left("mean of empty list!")
  else
    Right(xs.sum / xs.length)
```

- Variant if we want to include more information on the error

```scala
def safeDev(x: Int, y: Int): Either[Exception, Int] =
  try Right(x/y)
  catch { case e: Exception => Left(e) }
```

- Can be further generalized into it's own function

```scala
def Try[A](a: => A): Either[Exception, A] =
  try Right(a)
  catch { case e: Exception => Left(e) }
```

- Example: Using `Either` to validate data

```scala
case class Person(name: Name, age: Age)
sealed class Name(val value: String)
sealed class Age(val value: Int)

def mkName(name: String): Either[String, Name] =
  if (name == "" || name == null) Left("Name is empty.")
  else Right(new Name(name))

def mkAge(age: Int): Either[String,Age] =
  if (age < 0) Left("Age is out of range.")
  else Right (new Age(age))

def mkPerson(name: String, age: Int): Either[String Person] =
  mkName(name).map2(mkAge(age))(Person(_,_))
```
