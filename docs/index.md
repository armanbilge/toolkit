# typelevel-toolkit

A toolkit of **great libraries** to start building **Typelevel** apps on JVM, Node.js, and Native!

Our very own flavour of the [Scala Toolkit].

## Overview

Typelevel Toolkit is a meta library that currently includes these libraries:

* [Cats] and [Cats Effect] for writing composable programs
* [FS2] for streaming, file I/O, and subprocesses
* [http4s Ember client] for HTTP requests
* [Circe] for JSON handling
* [fs2-data] for CSV handling
* [Decline Effect] for developing a CLI app
* [MUnit Cats Effect] for writing tests

It is published for Scala 2.13 and 3.2+ and also fully supports Scala.js 1.13+ and Scala Native 0.4.

### Scala JVM

@:select(build-tool)
@:choice(scala-cli)
```scala
//> using toolkit "typelevel:@VERSION@"
```
@:choice(sbt)
```scala
libraryDependencies ++= Seq(
  "org.typelevel" %% "toolkit" % "@VERSION@",
  "org.typelevel" %% "toolkit-test" % "@VERSION@" % Test
)
```
@:@

#### GraalVM Native Image

The `--no-fallback` option is required ([further reading][Native Image]).

@:select(build-tool)
@:choice(scala-cli)
The `package` command [supports Native Image](https://scala-cli.virtuslab.org/docs/cookbooks/native-images/).
```
scala-cli --power package --native-image myscript.scala -- --no-fallback
```
@:choice(sbt)
Please use the [sbt-native-image plugin](https://github.com/scalameta/sbt-native-image).
```scala
enablePlugins(NativeImagePlugin)
nativeImageOptions += "--no-fallback"
```
@:@

### Scala.js

You should enable the [CommonJS module kind](https://www.scala-js.org/doc/project/module.html) so that libraries such as FS2 can import Node.js standard library modules.

@:select(build-tool)
@:choice(scala-cli)
Scala CLI has [excellent support for Scala.js](https://scala-cli.virtuslab.org/docs/guides/scala-js).
```scala
//> using toolkit "typelevel:@VERSION@"
//> using platform "js"
//> using jsModuleKind "common"
```
@:choice(sbt)
Please use the [sbt-scalajs plugin](https://www.scala-js.org/doc/project/).
```scala
enablePlugins(ScalaJSPlugin)
scalaJSUseMainModuleInitializer := true
scalaJSLinkerConfig ~= (_.withModuleKind(ModuleKind.CommonJSModule)) 
libraryDependencies ++= Seq(
  "org.typelevel" %%% "toolkit" % "@VERSION@",
  "org.typelevel" %%% "toolkit-test" % "@VERSION@" % Test
)
```
@:@

### Scala Native

@:select(build-tool)
@:choice(scala-cli)
Scala CLI has [excellent support for Scala Native](https://scala-cli.virtuslab.org/docs/guides/scala-native).
```scala
//> using toolkit "typelevel:@VERSION@"
//> using platform "native"
```
@:choice(sbt)
Please use the [sbt-scala-native plugin](https://scala-native.org/en/stable/user/sbt.html).
```scala
enablePlugins(ScalaNativePlugin)
libraryDependencies ++= Seq(
  "org.typelevel" %%% "toolkit" % "@VERSION@",
  "org.typelevel" %%% "toolkit-test" % "@VERSION@" % Test
)
```
@:@

## Quick Start Example
@:select(scala-version)
@:choice(scala-3)
```scala mdoc:reset:silent
//> using lib "org.typelevel::toolkit::@VERSION@"

import cats.effect.*

object Hello extends IOApp.Simple:
  def run = IO.println("Hello toolkit!")
```
@:choice(scala-2)
```scala mdoc:reset:silent
//> using lib "org.typelevel::toolkit::@VERSION@"

import cats.effect._

object Hello extends IOApp.Simple {
  def run = IO.println("Hello toolkit!")
}
```
@:@

[Scala CLI]: https://scala-cli.virtuslab.org
[Scala Toolkit]: https://github.com/scala/toolkit
[Cats]: https://typelevel.org/cats
[Cats Effect]: https://typelevel.org/cats-effect
[FS2]: https://fs2.io/#/
[FS2 I/O]: https://fs2.io/#/io
[fs2-data]: https://fs2-data.gnieh.org/documentation/csv/
[http4s Ember Client]: https://http4s.org/v0.23/docs/client.html
[Circe]: https://circe.github.io/circe/
[Decline Effect]: https://ben.kirw.in/decline/effect.html
[MUnit Cats Effect]: https://github.com/typelevel/munit-cats-effect
[Native Image]: https://github.com/typelevel/cats-effect/issues/3051#issuecomment-1167026949
