[//]: # (title: Readability)

This chapter discusses considerations about [API consistency](#api-consistency) and provides the following recommendations:
* [Use a builder DSL](#use-a-builder-dsl)
* [Use constructor-like functions where applicable](#use-constructor-like-functions-where-applicable)
* [Use member and extension functions appropriately](#use-member-and-extension-functions-appropriately)
* [Avoid using Boolean arguments in functions](#avoid-using-boolean-arguments-in-functions)

## API consistency

A consistent and well-documented API is crucial for a good development experience. The same is valid for argument order, 
overall naming scheme, and overloads. Also, it's worth documenting all conventions.

For example, if one of your methods accepts `offset` and `length` as parameters, then so should other methods, 
instead of, for example, accepting `startIndex` and `endIndex`. Parameters like these are most likely of `Int` or `Long` 
type, and thus it's very easy to confuse them.

The same works for parameter order: Keep it consistent between methods and overloads. Otherwise, users of your library 
might incorrectly guess the order they should pass arguments in.

Here is an example that preserves the parameter order and uses consistent naming:

```kotlin
fun String.chop(length: Int): String = substring(0, length)
fun String.chop(length: Int, startIndex: Int) =
    substring(startIndex, length + startIndex)
```

If you have many lookalike methods, name them consistently and predictably. This is how the `stdlib` API works: 
There are methods `first()` and `firstOrNull()`, `single()` and `singleOrNull()`, and so on. It's clear from their names 
that they are all pairs, and some of them might return `null` while others might throw an exception.

## Use a builder DSL

["Builder"](https://en.wikipedia.org/wiki/Builder_pattern#:~:text=The%20builder%20pattern%20is%20a,Gang%20of%20Four%20design%20patterns) 
is a well-known pattern in development. It allows you to build a complex entity not in a single expression, but gradually 
while getting more information. When you need to use a builder, it's better to write it using a builder DSL, which 
is binary-compatible and more idiomatic.

A canonical example of a Kotlin builder DSL is `kotlinx.html`. Consider the following example:

```kotlin
header("modal-card-head") {
    p("modal-card-title") {
        +book.book.name
    }
    button(classes = "delete") {
        attributes["aria-label"] = "close"
        attributes["_"] = closeModalScript
    }
}
```

It could be implemented as a traditional builder, but that would be considerably more verbose:

```kotlin
headerBuilder()
    .addClasses("modal-card-head")
    .addElement(
        pBuilder()
            .addClasses("modal-card-title")
            .addContent(book.book.name)
            .build()
    )
    .addElement(
        buttonBuilder()
            .addClasses("delete")
            .addAttribute("aria-label", "close")
            .addAttribute("_", closeModalScript)
            .build()
    )
    .build()
```

This implementation has too many details that you don't necessarily need to know, and it requires you to build 
each entity at the end.

The situation gets even worse if you need to generate a builder's content dynamically in a loop. In this scenario, 
you have to instantiate a variable and dynamically overwrite it:

```kotlin
var buttonBuilder = buttonBuilder()
    .addClasses("delete")
for ((attributeName, attributeValue) in attributes) {
    buttonBuilder = buttonBuilder.addAttribute(attributeName, attributeValue)
}
buttonBuilder.build()
```

Inside the builder DSL, you can directly call a loop and all necessary DSL calls:

```kotlin
div("tags") {
    for (genre in book.genres) {
        span("tag is-rounded is-normal is-info is-light") {
            +genre
        }
    }
}
```

Keep in mind that inside curly braces it's impossible to check at compile time whether you have set all the required 
attributes. To avoid this, pass required arguments as function arguments, not as builder's properties. For example, 
if you want `href` to be a mandatory HTML attribute, your function will look like:

```kotlin
fun a(href: String, block: A.() -> Unit): A
```

And not just:

```kotlin
fun a(block: A.() -> Unit): A
```

> Builder DSLs are [backward-compatible](jvm-api-guidelines-backward-compatibility.md) as long as you don't delete 
> anything from them. Typically this isn't a problem, because most developers only add more properties to their builder 
> classes over time.
>
{type="note"}

## Use constructor-like functions where applicable

Sometimes, you can simplify your API's appearance by using constructor-like functions. A constructor-like function is 
a function whose name starts with a capital letter, so it looks like a constructor. This approach can make your library 
easier to understand.

Suppose you want to introduce an [Option type](https://en.wikipedia.org/wiki/Option_type) in your library:

```kotlin
sealed interface Option<T>
class Some<T : Any>(val t: T) : Option<T>
object None : Option<Nothing>
```

You can define implementations of all the `Option` interface methods – `map()`, `flatMap()`, and so on. However, 
each time your API users create such an `Option`, they must write extra logic to check what they create. For example:

```kotlin
fun findById(id: Int): Option<Person> {
    val person = db.personById(id)
    return if (person == null) None else Some(person)
}
```

To save your users from having to write the same check each time, you can add just one line to your API:

```kotlin
fun <T> Option(t: T?): Option<out T & Any> =
    if (t == null) None else Some(t)

// Usage of the code above:
fun findById(id: Int): Option<Person> = Option(db.personById(id))
```

Now, creating a valid `Option` is as simple as can be: Just call `Option(x)` and you have a null-safe, purely functional 
Option idiom.

Another use case for using a constructor-like function is when you need to return "hidden" things, such as a private 
instance or an internal object. For example, let's look at a method from the standard library:

```kotlin
public fun <T> listOf(vararg elements: T): List<T> =
    if (elements.isNotEmpty()) elements.asList() else emptyList()
```

In the code above, `emptyList()` returns the following:

```kotlin
internal object EmptyList : List<Nothing>, Serializable, RandomAccess
```

You can write a constructor-like function to lower the [cognitive complexity](jvm-api-guidelines-introduction.md#cognitive-complexity)
of your code and reduce the size of your API:

```kotlin
fun <T> List(): List<T> = EmptyList

// Usage of the code above:
public fun <T> listOf(vararg elements: T): List<T> =
    if (elements.isNotEmpty()) elements.asList() else List()
```

## Use member and extension functions appropriately

Write only the very core of the API as [member functions](functions.md#member-functions), and everything else as 
[extension functions](extensions.md#extension-functions). This will help you clearly show to the reader what is 
the core functionality and what isn't.

For example, consider the following class for a graph:

```kotlin
class Graph {
    private val _vertices: MutableSet<Int> = mutableSetOf()
    private val _edges: MutableMap<Int, MutableSet<Int>> = mutableMapOf()

    fun addVertex(vertex: Int) {
        _vertices.add(vertex)
    }

    fun addEdge(vertex1: Int, vertex2: Int) {
        _vertices.add(vertex1)
        _vertices.add(vertex2)
        _edges.getOrPut(vertex1) { mutableSetOf() }.add(vertex2)
        _edges.getOrPut(vertex2) { mutableSetOf() }.add(vertex1)
    }

    val vertices: Set<Int> get() = _vertices
    val edges: Map<Int, Set<Int>> get() = _edges
}
```

This class contains a bare minimum of vertices and edges as private variables, functions to add vertices and edges, and 
accessor functions that return an immutable representation of the current state.

You can add all the remaining functionality outside the class:

```kotlin
fun Graph.getNumberOfVertices(): Int = vertices.size
fun Graph.getNumberOfEdges(): Int = edges.size
fun Graph.getDegree(vertex: Int): Int = edges[vertex]?.size ?: 0
```

Only properties, overrides and accessors should be members.

## Avoid using Boolean arguments in functions

Ideally, a reader should be able to tell the purpose of a function argument just by reading code. With `Boolean` 
arguments, however, this is almost impossible to do, especially if you're not using an IDE (for example, if you're 
reviewing the code in a version control service). Using [named arguments](functions.md#named-arguments) can help clarify 
the purpose of arguments, but for now there is no way to force developers to use them in IDEs. Another option is 
to create a function that contains the action of the `Boolean` argument and give this function a descriptive name.

For example, in the standard library there are two functions for `map()`:

```kotlin
fun map(transform: (T) -> R): List<R>

fun mapNotNull(transform: (T) -> R?): List<R>
```    

It was possible to add something like `map(filterNulls: Boolean)` and write code like this:

```kotlin
listOf(1, null, 2).map(false) { it.toString() }
```

From reading this code, it's difficult to infer what `false` refers to. However, if you use the `mapNotNull()` function, 
readers will be able to understand the logic at a glance:

```kotlin
listOf(1, null, 2).mapNotNull { it.toString() } 
```

## Use typed units

Use typed units for anything having units of measure. It will preserve your users from order of magnitude mistakes.

Let's look at the following function:

```kotlin
fun sleep(time: Long) {
    // implementation
}
```

It has an obvious flaw: we don't know in which units should we pass an argument there. In `bash` it would be seconds, in `Java` probably milliseconds, somewhere it could be even nanoseconds. It's easy to make a mistake for 3 orders of magnitude!

Of course we can write a self-ducumenting code:

```kotlin
fun sleep(millis: Long){
    // implementation
}
```

But this code is still imperfect. On the reader side it will require tools to understand the code. They will need to see an inlay hint or something similar to understand what's going on and even deeper analysis to underdtand if there's an error there.

Instead consider adding a unit measure as an additional argument, like this:

```kotlin
fun sleep(amount: Long, unit: TimeUnit){
    // implementation
}
```

Here we fix all the problems at once:

1. Now the reader of the consuming code clearly sees what is passed and where, and in which units
2. The probability to make n order-of-magnitude error is significantly smaller
3. On the library side for you it's easier to implement conversion rules

## What's next?

Learn about APIs':
* [Predictability](jvm-api-guidelines-predictability.md)
* [Debuggability](jvm-api-guidelines-debuggability.md)
* [Backward compatibility](jvm-api-guidelines-backward-compatibility.md)
