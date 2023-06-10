# Cats and Cats Effect

Cats and Cats Effect are the backbone of all Typelevel Scala applications. Together, they enable us to build and compose programs, and then execute these programs on a high-performance runtime.

## Writing a Cats Effect program

To write a Cats Effect program we work with a special datatype `IO`. More specifically, `IO` is a building block for creating programs. You can combine it with other `IO`s to make another, “bigger” `IO` that is a program with more steps in it.

Let’s look at an example. Here is an `IO` program. If we were to run this program, it would print `"hello"`.

```scala
val helloProgram: IO[Unit] = IO.println("hello")
```

Here is a similar program, that prints `"world"` when run.

```scala
val worldProgram: IO[Unit] = IO.println("world")
```

What if we want a program that, when run, prints `"hello"` and then prints `"world"`? Let’s combine our `helloProgram` and our `worldProgram` using the `*>` “and then” operator. Now, we have a `helloWorldProgram`!

```scala
val helloWorldProgram: IO[Unit] = helloProgram *> worldProgram
```

We can visualize how the `IO`s are combined into a larger program. This is because `IO` is actually a datatype.

<pre class="mermaid">
flowchart LR
  subgraph "helloWorldProgram: IO[Unit]"
    direction LR
    hello["IO.println(#quot;hello#quot;)"] -- "*>" --> world["IO.println(#quot;world#quot;)"];
  end
</pre>

We can continue combining the `helloWorldProgram` with other `IO`s to add more steps and build an even bigger `IO` program.

```scala
val helloWorldByeProgram: IO[Unit] = helloWorldProgram *> IO.println("kbye")
```

<pre class="mermaid">
flowchart LR
  subgraph "helloWorldByeProgram: IO[Unit]"
    direction LR
    subgraph helloWorld["helloWorldProgram: IO[Unit]"]
      direction LR
      hello["IO.println(#quot;hello#quot;)"] -- "*>" --> world["IO.println(#quot;world#quot;)"];
    end
    helloWorld["helloProgram: IO[Unit]"] -- "*>" --> bye["IO.println(#quot;kbye#quot;)"];
  end
</pre>

**So far all that we have done is create data**: a description of a program that, when run, will print `"hello"`, then `"world"`, then `"kbye"`. In the next section we will see how we can bring this data to life as a running application.

@:callout(warning)

In other programming languages, you may be used to adding a new step to your program by adding a new line.

```scala
println("hello")
println("world")
println("kbye")
```

**This will not work with `IO` programs**. Instead of building up a source file by adding lines, you build up an `IO` by combining `IO`s with `*>` and other operators.

```scala
IO.println("hello") *> IO.println("world") *> IO.println("kbye")
```

@:@

## Running a Cats Effect program

We are done building our program! To run it, we will create an `IOApp` and assign our `IO` program to `val run`. This hands the program data over to the Cats Effect runtime, which will step through and actually execute it for us.

@:select(scala-version)
@:choice(scala-3)
```scala
//> using toolkit typelevel:@VERSION@

import cats.effect.IO
import cats.effect.IOApp

object HelloApp extends IOApp.Simple:
  val helloProgram: IO[Unit] = IO.println("hello")
  val worldProgram: IO[Unit] = IO.println("world")
  val helloWorldProgram: IO[Unit] = helloProgram *> worldProgram
  val helloWorldByeProgram: IO[Unit] = helloWorldProgram *> IO.println("kbye")

  // give our program to the Cats Effect runtime for execution
  val run: IO[Unit] = helloWorldByeProgram
```
@:choice(scala-2)
```scala
//> using toolkit typelevel:@VERSION@

import cats.effect.IO
import cats.effect.IOApp

object HelloApp extends IOApp.Simple {
  val helloProgram: IO[Unit] = IO.println("hello")
  val worldProgram: IO[Unit] = IO.println("world")
  val helloWorldProgram: IO[Unit] = helloProgram *> worldProgram
  val helloWorldByeProgram: IO[Unit] = helloWorldProgram *> IO.println("kbye")

  // give our program to the Cats Effect runtime for execution
  val run: IO[Unit] = helloWorldProgram
}
```
@:@

Save this as `hello.scala` and run it with [Scala CLI](https://scala-cli.virtuslab.org/).
```
$ scala-cli hello.scala
hello
world
kbye
```

@:callout(info)

Cats Effect has specialized runtimes for the Java Virtual Machine, Node.js, and standalone Native executables. As long as you build your `IO` program using only Typelevel libraries, Cats Effect will run it on any supported platform with identical semantics and behavior.

```
$ scala-cli --js hello.scala
$ scala-cli --native hello.scala
```

@:@

## Passing results between steps of your program

So far, all of our programs have had type `IO[Unit]`. These are programs that complete with a value of type `Unit` aka “void”. As this result offers minimal information, we have been content to combine our `IO`s using the `*>` “and then” operator which simply discards the result of the left-hand-side.

In a more complex application, our `IO` programs can produce interesting results. We may want subsequent steps in our program to use those results so we will need a way to pass them along.

Let’s look at an example. Consider the program `IO.realTime: IO[FiniteDuration]`. This is an `IO` program that, when run, returns the current time (reported as the number of microseconds since the [Unix epoch](https://en.wikipedia.org/wiki/Unix_time)).

Now suppose we want to print this time using the `IO.println(...)` program. To combine `IO` programs while passing results from left-to-right we use the `flatMap` operator.

@:select(scala-version)
@:choice(scala-3)
```scala
val printUnixTime: IO[Unit] =
  IO.realTime.flatMap: time =>
    IO.println(time)
```
@:choice(scala-2)
```scala
val printUnixTime: IO[Unit] =
  IO.realTime.flatMap { time =>
    IO.println(time)
  }
```
@:@

Here, `flatMap` binds the result of the first step `IO.realTime` to the `time` parameter. Now that we have the `time`, we can use it to build the next step `IO.println(time)`. Finally, `flatMap` combines these two steps into a new `IO` which we call `printUnixTime`.

<pre class="mermaid">
flowchart LR
  subgraph "printUnixTime: IO[Unit]"
    direction LR
    realTime["IO.realTime"] -- "flatMap: time" --> println["IO.println(time)"];
  end
</pre>

If we make an `IOApp` and run this program then we will get an output like this.

```
1686438032685889 microseconds
```

Do you _know_ what time it is now?

We need something more human-readable but `IO.realTime` gives the result in an annoying format. So we will use the `map` operator to transform the `IO[FiniteDuration]` program into a `IO[Instant]` program. Then we will use `flatMap` as above to print the `Instant`.

@:select(scala-version)
@:choice(scala-3)
```scala
import java.time.Instant

val humanTime: IO[Instant] =
  IO.realTime.map: unixTime =>
    Instant.EPOCH.plusNanos(unixTime.toNanos)

val printHumanTime: IO[Unit] =
  humanTime.flatMap: time =>
    IO.println(time)
```
@:choice(scala-2)
```scala
import java.time.Instant

val humanTime: IO[Instant] =
  IO.realTime.map { unixTime =>
    Instant.EPOCH.plusNanos(unixTime.toNanos)
  }

val printHumanTime: IO[Unit] =
  humanTime.flatMap { time =>
    IO.println(time)
  }
```
@:@

If we study the visual, we see that our program still only has two fundamental steps: the `IO.realTime` and the `IO.println(...)`. The `map` operator transforms the first step so that its result is more useful. Then the `flatMap` operator combines the two steps into our final program.

<pre class="mermaid">
flowchart LR
  subgraph "printHumanTime: IO[Unit]"
    direction LR
    subgraph humanTime["humanTime: IO[Instant]"]
      direction LR
      realTime["IO.realTime"] -- "map: unixTime to Instant" --> mapped[ ];
      style mapped width:0px;
    end
    humanTime -- "flatMap: time" --> println["IO.println(time)"];
  end
</pre>

<script type="module" src="mermaid.js"></script>

When we make an `IOApp` for `printHumanTime` we get this, could be worse!
```
2023-06-10T23:33:21.399386Z
```
