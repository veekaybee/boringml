---
title: build.sbt and build.Scala
weight: 2
bookToc: false
---

# Building Scala code

SBT, along with Maven, is a default way to build Scala applications. `build.sbt` is the file that defines how your project is built, but sometimes you'll also see `build.scala` files in specific projects. 

`build.scala` is the more advanced version of the `build.sbt` file, and often is used for more complicated projects. 

[Here's an example](https://stackoverflow.com/questions/18000103/what-is-the-difference-between-build-sbt-and-build-scala) of the difference between .sbt and .scala build files: 

build.sbt
```
name := "hello"

version := "1.0"
```

build.scala
```
import sbt._
import Keys._

object Build extends Build {
  lazy val root = Project(id = "root", base = file(".")).settings(
    name := "hello",
    version := "1.0"      
  )
}
```



From [the official docs]( https://www.scala-sbt.org/1.x/docs/Organizing-Build.html#When+to+use++files) 

```
The recommended approach is to define most settings in a multi-project build.sbt file, and using project/*.scala files for task implementations or to share values, such as keys. The use of .scala files also depends on how comfortable you or your team are with Scala.
```

Here's [another great doc on sbt vs Scala files.](https://alvinalexander.com/scala/sbt-how-to-use-build.scala-instead-of-build.sbt/) 

An important note is that this functionality is deprecated as of [sbt 0.13.12](https://github.com/sbt/sbt/pull/2530). Here's more about [how these two files work together](https://www.scala-sbt.org/0.13/docs/Full-Def.html)