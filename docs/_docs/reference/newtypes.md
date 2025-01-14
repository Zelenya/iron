---
title: "Creating New Types"
---

# Creating New Types

You can create no-overhead new types like [scala-newtype](https://github.com/estatico/scala-newtype) in Scala 2 with Iron.

## Opaque types

The aliased type of an [opaque type](https://docs.scala-lang.org/scala3/book/types-opaque-types.html) is only known in its definition file. It is not considered like a type alias outside of it:

```scala
opaque type Temperature = Double :| Positive
```

```scala
val x: Double :| Positive = 5
val temperature: Temperature = x //Error: Temperature expected, got Double :| Positive
```

Such encapsulation is especially useful to avoid mixing different domain types with the same refinement:

```scala
opaque type Temperature = Double :| Positive
opaque type Moisture = Double :| Positive
```

```scala
case class Info(temperature: Temperature, moisture: Moisture)

val temperature: Temperature = ???
val moisture: Moisture = ???

Info(moisture, temperature) //Compile-time error
```

But as is, you cannot create an "instance" of your opaque type outside of its definition file. You need to add methods yourself like:

```scala
opaque type Temperature = Double :| Positive

object Temperature:

  def apply(x: Double :| Positive): Temperature = x
```

## RefinedTypeOps

Iron provides a convenient trait called `RefinedTypeOps` to easily add smart constructors to your type:

```scala
opaque type Temperature = Double :| Positive
object Temperature extends RefinedTypeOps[Temperature]
```

```scala
val temperature = Temperature(15) //Compiles
println(temperature) //15

val positive: Int :| Positive = 15
val tempFromIron = Temperature.fromIronType(positive)
```

### Runtime refinement

`RefinedTypeOps` supports [all refinement methods](refinement.md) provided by Iron:

```scala
val unsafeRuntime: Temperature = Temperature.applyUnsafe(15)
val option: Option[Temperature] = Temperature.option(15)
val either: Either[String, Temperature] = Temperature.either(15)
```

Constructors for other modules exist:

```scala
val zioValidation: Temperature = Temperature.validation(15)
```

Note: all these constructors are inline. They don't bring any overhead:

```scala
val temperature: Temperature = Temperature(15)
val unsafeRuntime: Temperature = Temperature.applyUnsafe(runtimeValue)
```

compiles to

```scala
val temperature: Double = 15
val unsafeRuntime: Double =
  if runtimeValue > 0 then runtimeValue
  else throw new IllegalArgumentException("...")
```

