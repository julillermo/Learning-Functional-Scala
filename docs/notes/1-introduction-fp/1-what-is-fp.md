# Intro

- FP -> we construct programs with pure functions (no side-effects)
- Kinds of side-effects:
  - Modify a variable
  - Setting field on an obj
  - Throwing an exception / handling error
  - Print to console / reading input
  - Read / Write files
  - Interacting with the screen

- Functional programming improves _modularity_
  - easier to test, resuse, parallelize, generalize, and debug

## Benefits of FP

- Example scala code w/ side-effects
  - The `cc.charge(cup.price)` is a side-effect. It interacts with the outside world even though the function was meant to return `Coffee`.
  - This makes tracing out the logic during debugging difficult.
  - The `buyCoffee` function is also difficult to reuse.
    - It charges per 1 coffee, so a payment transaction (likely an API call) is made 8 times if we order 8 cups

```scala
class Cafe {
  def buyCoffee(cc: CreditCard): Coffee = {
    val cup = new Coffee()
    cc.charge(cup.price)
    cup
  }:
}
```

### A functional solution: remove the side-effects

- Return a `Charge` object that can be collated and used elsewhere alongside the originallly returned `Coffee` object
- The responsibility of working on the `Charge` is correctly removed from the "buying" logic and passed on to somewhere more appropriate.
  - This opens the possibility of a set of ways to work with `Charge` related logic

```scala
class Cafe {
  def buyCoffee(cc: CreditCard): (Coffee, Charge) = {
    val cup = new Coffee()
    (cup, Charge(cc, cup.price))
  }

  def buyCoffees(cc: CreditCard, n: Int): (List[Coffee] Charge) = {
    val purchaes: List[(Coffee, Charge)] = List.fill(n)(buyCoffee(cc))
    val (coffees, charges) = purchases.unzip
    (coffees, charges.reduce((c1,c2) => c1.combine(c2)))
  }
}

case class Charge (cc: CreditCard, amount: Double) {
  def combine(other: Charge): Charge =
    if (cc == other.cc)
      Charge(cc, amount + other.amount)
    else
      throw new Exception("Can't combine charges to different cards")\

  // I believe the below if a functional example
  def coalesce(charges: List[Charge]): List[Charge] =
    carges.groupBy(_.cc).values.map(_.reduce(_ combine _)).toList
}
```

## Exactly what is a (pure) function?

- My simplification of the more formal definition of functional programming
  - _The output is exactly determined solely according to the input_
  - Think of math operations. They're pure functions! Pure functions must also evaluate into a result

## Referential transparency, purity, and the substitution model

- _substitution model_
  - If we interpret code as a combination of mathemtical expressions (in one equation), we should be able to swap out values with their formuals and vice versa
  - Additionally, each instances of a code logic should be expected to always return the same value. If there are 2 places where code is typed out the same, the output should be the same as well

```scala
val x = new StringBuilder("Hello")

val r1 = x.append(", World").toString
// result: Hello, World
val r2 = x.append(", World").toString
// result: Hello, World, World
```

- Since `r1` and `r2` do not agree, the `.append()` method is NOT a pure function
  - In FP, "... we need not mentally simulate sequences of state updates to understand a block of code." We only need _local reasoning_. They're essentialy _composable_
