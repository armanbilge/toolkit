# Cats and Cats Effect

Cats and Cats Effect are the backbone of Typelevel Scala applications. Together, they enable us to build and compose programs, and then execute these programs on a high-performance runtime.

### Writing a Cats Effect program

To write a Cats Effect program we work with a special datatype `IO`. More specifically, `IO` is a sort of building block for creating programs. You can combine it with other `IO`s to make another, “bigger” `IO` that is just a program with more steps in it.

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

@:callout(warning)

In other programming languages, you may be used to adding a new step to your program by adding a new line.

```scala
println("hello")
println("world")
```

**This will not work with `IO` programs**. Instead of building up a source file by adding lines, you build up an `IO` by combining `IO`s with `*>` and other operators.

```scala
IO.println("hello") *> IO.println("world")
```

@:@

### Running a Cats Effect program

We are done building our program! To run it, we will create an `IOApp` and assign our `IO` program to `val run`. This hands it over to the Cats Effect runtime, which will execute it for us. 

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

  val run: IO[Unit] = helloWorldProgram
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

  val run: IO[Unit] = helloWorldProgram
}
```
@:@

Save this as `hello.scala` and run it with Scala CLI.
```
$ scala-cli hello.scala
hello
world
```

@:callout(info)

Cats Effect has specialized runtimes for the Java Virtual Machine, Node.js, and standalone Native executables. As long as you build your `IO` program using only Cats Effect and other Typelevel libraries, Cats Effect will run it on any supported platform with identical semantics and behavior.

```
$ scala-cli --js hello.scala
$ scala-cli --native hello.scala
```

@:@
