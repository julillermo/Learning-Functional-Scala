# External effects and I/O

- examples of external effects:
  - reading from databases
  - writing to files

## Factoring effects

- sample program with side effects (printing to terminal)

```scala
case class Player(name: String score: Int)

def contest(p1: Player, p2: Player): Unit =
  if (p1.score > p2.score)
    println(s"${p1.name} is the winner!")
  else if (p2.score > p1.score)
    println(s"${p2.name} is the winner!")
  else
    println("It's a draw.")
```

- It's always possible to factor the pure part of the code separate from the side-effect. Refer to below:
  - We separated out the responsibilty of determining the winner and declaring the winner

```scala
def winner(p1: Player, p2: Player): Option[Player] =
  if (p1.score > p2.score) Some(p1)
  else if (p1.score < p2.score) Some(p2)
  else None
def contenst(p1: Player, p2: Player2): Unit = winner(p1,p2) match {
  case Some(Player(name,_)) => prinln(s"$name is the winner!")
  case None => println("It's a draw.")
}
```

- Further refactoring:
  - Technically `contenst` had 2 responsibilities: computing the message to display + printing to the console

> inside every function with side effects is a pure function waiting to get out.

```scala
def winner(p1: Player, p2: Player): Option[Player] =
  if (p1.score > p2.score) Some(p1)
  else if (p1.score < p2.score) Some(p2)
  else None

def winnerMsg(p: Option[Player]): String = p map {
  case Player(name, _) => s"$name is the winner!"
} getOrElse "It's a draw."

// The side-effect, println, is now segregated at the outermost part of the program
def contest(p1: Player, p2: Player): Unit =
  println(winnerMsg(winner(p1,p2)))
```

- Give an impure function `f: A => B`, we should be able to break this down into
  - A _pure_ function of type A => D, where `D` is a _description_ of the result
  - An _impure_ function of type D => B, the _interpreter_ of the descriptions

## A simple `IO` type

- The `println` command is also doing more than 1 thing underneath the hood. We can separate them out using a type known as `IO`
- In the following implementation the `contenst` function is now _pure_ (returns an `IO` value)
  - `IO` value -> An action that needs to take place
  - The `contest` function produces an _effect_ or is _effectful_, but it's the `run` function that interprets the side-effect

```scala
trait IO { self =>
  def run: Unit
  def ++(io: IO): IO = new IO {
    def run = { self.run; io.run }
  }
}

object IO {
  def empty: IO = new IO { def run = () }
}

def winner(p1: Player, p2: Player): Option[Player] =
  if (p1.score > p2.score) Some(p1)
  else if (p1.score < p2.score) Some(p2)
  else None

def winnerMsg(p: Option[Player]): String = p map {
  case Player(name, _) => s"$name is the winner!"
} getOrElse "It's a draw."

// Our explicit representation of the println command
def PrintLine(msg: String): IO =
  new IO { def run  = println(msg) }

def contest(p1: Player, p2: Player): IO =
  PrintLine(winnerMsg(winner(p1,p2)))
```

- Note from the above code snippet that `IO` formas a `Monoid` (`empty` serves as identity and `++` serves as the associative operatioon)
  - If we have a `List[IO]`, this means we can reduce the list into a single `IO`
  - Associativity grants use freedom of folding leftward vs folding rightward.

### Handling input effects

- The above `IO` only represents output effects, but not input.
- Extend our `IO` type to allow _input_, it now becomes a `Monad`

```scala
sealed trait IO[A] { self =>
  def run: A
  def map[B](f: A => B): IO[B] =
    new IO[B] { def run = f(self.run) }
  def flatMap[B](f: A => IO[B]): IO[B] =
    new IO[B] { def run = f(self.run).run }
}

object IO extends Monad[IO] {
  def unit[A](a: => A): IO[A] = new IO[A] { def run = a }
  def flatMap[A,B](fa: IO[A])(f: A => IO[B]) = fa flatMap f
  def apply[A](a: => A): IO[A] = unit(a)
}

def ReadLine: IO[String] = IO { readLine }
def PrintLine(msg: String): IO[Unit] = IO { println(msg) }

def fahrenheitToCelsius(f: Double): Double =
  (f - 32) * 5.0/9.0

def converter: IO[Unit] = for {
  _ <- PrintLine("Enter a temperature in degrees Fahrenheit: ")
  d <- ReadLine.map(_.toDouble)
  _ <- PrintLine(fahrenheightToCelsius(d).toString)
} yield()
```

- Some other example usage of IO:
  - `val echo = ReadLine.flatMap(PrintLine)`
    - `IO[Unit]` unit read from console and echo
  - `val readInt = ReadLine.map(_.toInt)`
    - `IO[Unit]` read from console and parse the int
  - `val readInt = readInt ** readInt`
    - `IO[(Unit,Unit)]` read and parse for int form console twice
  - `replicateM(10)(ReadLine)`
    - `IO[List[String]]` read 10 lines and store as list

### Benefits and drawbacks of the simple IO type

> ... when programming _within_ the `IO` monad, we have many of the same difficulties as we would in ordinary imperative programming, which ahs motivated functional programmers to look for more composable ways of describing effectful programs. Nonetheless, our `IO` monad does provide some real benefits:

- `IO` computations are ordinary `values`: can be placed in lists, passed around, created, and operated on, etc.
- Can allow for a more complex interpreter than the `run` example above

- Limitations of the custom `IO` implementation at the moment
  - Can lead into `StackOverflowError`
  - No way to perform concurrency or asynchronous operations

## Avoiding the `StackOverflowError`

- Consider the following:
  - `val p = IO.forever(PrintLine("Still going..."))`

### Reifying control flow as data costrutors

- skimmed
- FWIU: adjusted how the flatmap was declared because the original one was continously creating objects?

### Trampolining: a general solution to stack overflow

- skipped

## A more nuanced `IO` type

- The implementation up to this point had the following limitations
  - Doesn't know the kinds of effects
    -No way to implement asynchronous operations

## Non-blocking and asynchronous I/O

- The last remaining problem to the custom implementation of the `IO` monad up to this point:

## A general-purpose IO type

-
