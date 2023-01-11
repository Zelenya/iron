---
title: "ZIO Support"
---

# ZIO Support

This module provides refinement methods using [ZIO Prelude](https://zio.dev/zio-prelude/functionalabstractions/)'s `Validation`.

Prelude's typeclass instances already work with [[IronType|io.github.iltotore.iron.IronType]] due to variance.

## Dependency

SBT:

```scala
libraryDependencies += "io.github.iltotore" %% "iron-zio" % "version"
```

Mill:

```scala
ivy"io.github.iltotore::iron-zio:version"
```

## Accumulative error handling

ZIO enables accumulative error handling via [Validation](https://zio.dev/zio-prelude/functionaldatatypes/validation/). The ZIO module provides a `refineValidation` method that uses this datatype to handle errors.

This method is similar to `refineEither` and `refineOption` defined in the core module.

The [User example](../reference/refinement.md) now looks like this:


```scala
type Username = Alphanumeric DescribedAs "Username should be alphanumeric"

type Age = Greater[0] DescribedAs "Age should be positive"

case class User(name: String :| Username, age: Int :| Age)

def createUser(name: String, age: Int): Validation[String, User] =
  Validation.validateWith(
    name.refineValidation[Username],
    age.refineValidation[Age]
  )(User.apply)

createUser("Iltotore", 18) //Success(Chunk(),User(Iltotore,18))
createUser("Il_totore", 18) //Failure(Chunk(),NonEmptyChunk(Username should be alphanumeric))
createUser("Il_totore", -18) //Failure(Chunk(),NonEmptyChunk(Username should be alphanumeric, Age should be positive))
```