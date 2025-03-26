[//]: # (title: Readability)

Creating a readable API involves more than just writing clean code.
It requires thoughtful design that simplifies integration and usage.
This section explores how you can enhance API readability by structuring your library with composability in mind,
utilizing domain-specific languages (DSLs) for concise and expressive setup, and using extension functions and properties
for clear and maintainable code.

## Prefer Explicit Composability

Libraries often provide advanced operators that allow for customization.
For example, an operation might permit users to supply their own data structures, networking channels, timers, or lifecycle observers.
However, introducing these customization options through additional function parameters can significantly increase the complexity of the API.

Instead of adding more parameters for customization, it's more effective to design an API where different behaviors can
be composed together.
For example, in the coroutine Flows API both [buffering](flow.md#buffering) and [conflation](flow.md#conflation) are implemented as separate functions.
These can be chained together with more basic operations like [`filter`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/filter.html) and [`map`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html), instead of each basic operation accepting parameters to control buffering and conflation.

Another example involves the [Modifiers API in Jetpack Compose](https://developer.android.com/develop/ui/compose/modifiers).
This allows Composable components to accept a single `Modifier` parameter that handles common customization options, such as padding, sizing, and background color.
This approach avoids the need for each Composable to accept separate parameters for these customizations,
streamlining the API and reducing complexity.

```kotlin
Box(
    modifier = Modifier
        .padding(10.dp)
        .onClick { println("Box clicked!") }
        .fillMaxWidth()
        .fillMaxHeight()
        .verticalScroll(rememberScrollState())
        .horizontalScroll(rememberScrollState())
) {
    // Box content goes here
}
```

## Use DSLs

A Kotlin library can significantly improve readability by providing a builder DSL.
Using a DSL allows you to concisely repeat domain-specific data declarations.
For example, consider the following sample from a Ktor-based server application:

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json(Json {
            prettyPrint = true
            isLenient = true
        })
    }
    routing {
        post("/article") {
            call.respond<String>(HttpStatusCode.Created, ...)
        }
        get("/article/list") {
            call.respond<List<CreateArticle>>(...)
        }
        get("/article/{id}") {
            call.respond<Article>(...)
        }
    }
}
```

This sets up an application, installing the `ContentNegotiation` plugin configured to use Json serialization, and sets up
routing so that the application responds to requests on various `/article` endpoints.

For a detailed description of creating DSLs, see [Type-safe builders](type-safe-builders.md).
The following points are worth noting in the context of creating libraries:

* The functions used in the DSL are builder functions, which take a lambda with receiver as the final parameter.
  This design allows these functions to be called without parentheses, making the syntax clearer.
  The lambda being passed can be used to configure the entity being created. In the example above the lambda passed to the `routing` function is used to configure the details of the routing.
* Factory functions that create instances of classes should have the same name as the return type and start with a capital letter.
  You can see this in the sample above with the creation of the `Json` instance.
  These functions may still take lambda parameters for configuration. For more information, see [Coding conventions](coding-conventions.md#function-names).
* As it's not possible to ensure that required properties have been set within the lambda supplied to a builder function
  at compile time, we recommend passing required values as function parameters.

Using DSLs to build objects not only improves readability but also improves backward compatibility,
and simplifies the documentation process. For example, take the following function:

```kotlin
fun Json(prettyPrint: Boolean, isLenient: Boolean): Json
```

This function could replace the `Json{}` DSL builder. However, the DSL approach has noticeable benefits:

* Backward compatibility is easier to maintain with the DSL builder than with this function, as adding new configuration options simply means adding new properties (or in other examples, new functions), which is a backward-compatible change, unlike changing the parameter list of an existing function.
* It also makes creating and maintaining documentation easier. You can document each property separately at its point of declaration, instead of having to document many parameters of a function, all in one place.

## Use extension functions and properties

We recommend using [extension functions and properties](extensions.md) to improve readability.

Classes and interfaces should define the core concept of a type.
Additional functionality and information should be written as extension functions and properties.
This makes it clear to the reader that the additional functionality can be implemented on top of the core concept,
and additional information can be calculated from the data in the type.

For example, the [`CharSequence`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-char-sequence/) type (which `String` also implements) only contains the most basic information and operators to access its contents:

```kotlin
interface CharSequence {
    val length: Int
    operator fun get(index: Int): Char
    fun subSequence(startIndex: Int, endIndex: Int): CharSequence
}
```

Functionality commonly associated with strings is mostly defined as extension functions, which can all be implemented on
top of the core concepts and basic APIs of the type:

```kotlin
inline fun CharSequence.isEmpty(): Boolean = length == 0
inline fun CharSequence.isNotEmpty(): Boolean = length > 0

inline fun CharSequence.trimStart(predicate: (Char) -> Boolean): CharSequence {
    for (index in this.indices)
        if (!predicate(this[index]))
           return subSequence(index, length)
    return ""
}
```

Consider declaring computed properties and normal methods as extensions.
Only regular properties, overrides, and overloaded operators should be declared as members by default.

## Avoid using the boolean type as an argument

Consider the following function:

```kotlin
fun doWork(optimizeForSpeed: Boolean) { ... }
```

If you were to provide this function in your API it could be invoked as:

```kotlin
doWork(true)
doWork(optimizeForSpeed=true)
```

In the first call it is impossible to infer what the boolean argument is for, unless you are reading the code in an IDE
with Parameter Name Hints enabled.
Using named arguments does clarify the intention, but there is no way to force your users to adopt this style.
Consequently, to improve readability, your code should not use boolean types as arguments.

Alternatively, the API could create a separate function specifically for the task controlled by the boolean argument.
This function should have a descriptive name that indicates what it does.

For example, the following extensions are available on the [`Iterable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-iterable/) interface:

```kotlin
fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R>
fun <T, R : Any> Iterable<T>.mapNotNull(
    transform: (T) -> R?
): List<R>
```

Instead of the single method:

```kotlin
fun <T, R> Iterable<T>.map(
    includeNullResults: Boolean = true, 
    transform: (T) -> R
): List<R>
```

Another good approach could be to use an `enum` class to define different operation modes.
This approach is useful if there are several modes of operation, or if you expect these modes to change over time.

## Use numeric types appropriately

Kotlin defines a set of numeric types that you may use as part of your API. Here's how to use them appropriately:

* Use the `Int`,` Long` and `Double` types as arithmetic types. They represent values with which calculations are performed.
* Avoid using arithmetic types for non-arithmetic entities. For example, if you represent an ID as a `Long`, your users
  might be tempted to compare IDs, on the assumption they are assigned in order.
  This could lead to unreliable or meaningless results, or create dependencies on implementations that could change without warning.
  A better strategy is to define a specialized class for the ID abstraction. You could use [Inline value classes](inline-classes.md) to build such abstractions without affecting performance. See the  [`Duration`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.time/-duration/) class for an example.
* The `Byte`, `Float` and `Short` types are memory layout types. They are used to restrict the amount of memory available
  for storing a value, such as in caches or when transmitting data over a network.
  These types should only be used when the underlying data reliably fits within that type, and calculations are not required.
* The unsigned integer types `UByte`, `UShort`, `UInt` and `ULong` should be used to utilize the full range of positive
  values available in a given format. They are suitable for scenarios requiring values beyond the range of signed types or
  for interoperability with native libraries. However, avoid using them in situations where the domain only requires [non-negative integers](unsigned-integer-types.md#non-goals).

## Next step

In the next part of the guide, you'll learn about consistency.

[Proceed to the next part](api-guidelines-consistency.md)
