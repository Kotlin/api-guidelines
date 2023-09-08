[//]: # (title: Predictability)

This chapter contains the following recommendations:
* [Use sealed interfaces](#use-sealed-interfaces)
* [Hide implementations with sealed classes](#hide-implementations-with-sealed-classes)
* [Validate your inputs and state](#validate-your-inputs-and-state)
  * [Validate inputs with the require() function](#validate-inputs-with-the-require-function)
  * [Validate state with the check() function](#validate-state-with-the-check-function)
* [Avoid arrays in public signatures](#avoid-arrays-in-public-signatures)
* [Avoid varargs](#avoid-varargs)

## Use sealed interfaces

Interfaces in your API are usually necessary when you need to have an abstraction from an implementation. If you have 
to use interfaces, consider using [sealed interfaces](sealed-classes.md). This is especially important if you don't want 
your API's users to extend your hierarchy.

> Remember that adding a new implementation to a sealed interface will immediately make a user's existing code invalid.
>
{type="warning"}

For example, JSON types can be of six types: object, array, number, string, boolean, and null. Creating a generic 
`interface JsonElement` can result in errors because a user can accidentally define a new implementation of 
`JsonElement`, which could break your code. Instead, you can make `interface JsonElement` _sealed_ and add 
an implementation for each type:

```kotlin
sealed interface JsonElement

class JsonNumber(val value: Number) : JsonElement
class JsonObject(val values: Map<String, JsonElement>) : JsonElement
class JsonArray(val values: List<JsonElement>) : JsonElement
class JsonBoolean(val value: Boolean) : JsonElement
class JsonString(val value: String) : JsonElement
object JsonNull : JsonElement
```

This approach helps you avoid mistakes on both the library and the client sides.

The key benefit of using sealed types comes into play when you use them in a `when` expression. If it's possible 
to verify that the statement covers all cases, you don't need to add an `else` clause to the statement:

```kotlin
fun processJson(json: JsonElement) = when (json) {
    is JsonNumber -> { /* Process as a number */ }
    is JsonObject -> { /* Process as an object */ }
    is JsonArray -> { /* Process as an array */ }
    is JsonBoolean -> { /* Process as a boolean */ }
    is JsonString -> { /* Process as a string */ }
    is JsonNull -> { /* Process as null */ }
    // `else` clause is not required because all the cases are covered
}
```

## Hide implementations with sealed classes

If you have a sealed interface in your API, it doesn't mean that you should expose all its implementations in your API, 
too. Minimizing is typically better. If you need to avoid leaky abstractions or want to prevent API users from extending 
your interfaces, consider using sealed classes or interfaces with your internal implementations, too.

For example, a library that works with different databases can have an interface of a database response like this:

```kotlin
sealed interface DBResponse {
    operator fun <T> get(columnName: String): Sequence<T>
}
```

Exposing implementations of this interface, such as `SQLiteResponse` or `MongoResponse`, to API users is 
a **leaky abstraction**, and it complicates the support of this API. In such a library, you might handle only your 
implementations of `DBResponse`. If a user passes their implementation of `DBResponse` into a library's method 
accepting responses, it can cause an error. Using sealed interfaces and classes prevents this.

## Validate your inputs and state

### Validate inputs with the require() function

It's possible to misuse an API. To help your users work with your API correctly, you should validate inputs
as early as possible with the [require()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/require.html) 
function.

For example, this is a simple library function that saves users to some external API:

```kotlin
fun saveUser(username: String, password: String) {
    api.saveUser(User(username, password))
}
```

You should perform validation on the function's arguments to make sure that everything behaves as expected. For example, 
check that `username` is unique and not empty, even if you have already defined these constraints in your database:

```kotlin
fun saveUser(username: String, password: String) {
    require(username.isNotBlank()) { "Username should not be blank" }
    require(api.usernameAvailable(username)) { "Username $username is already taken" }
    require(password.isNotBlank()) { "Password should not be blank" }
    require(password.length > 6) { "Password should contain at least 7 letters" }
    require(
        /* Some complex check */
    ) { "..." }

    api.saveUser(User(username, password))
}
```

This way you ensure that your user doesn't need to dig into complex stack traces that lead to the database. In the event 
of an exception, it will be an `IllegalArgumentException` with a meaningful message, not a generic database exception.

> If you have implemented input validation, you should also document these checks.
>
{type="tip"}

### Validate state with the check() function

The same recommendations apply to checking the internal state. The most obvious example is `InputStream` because you 
can't read from a closed input stream.

Consider the class `InputStream` with a `readByte()` method and its usage:

```kotlin
class InputStream : Closeable {
    private var open = true
    fun readByte(): Byte { /* Read and return one byte */ }
    override fun close() {
        // Dispose of the underlying resource
        open = false
    }
}

fun readTwoBytes(inputStream: InputStream): Pair<Byte, Byte> {
    val first = inputStream.use { it.readByte() }
    val second = inputStream.readByte()
    return Pair(first, second)
}
```

The `readTwoBytes()` method has to throw an `IllegalStateException` because [`use{}`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/use.html) 
closes a `Closeable` input stream, and a user shouldn't be able to read from a closed stream. To implement this, modify 
the code of the `readByte()` function:

```kotlin
fun readByte(): Byte {
    check(open) { "Can't read from the already closed stream" }
    // Read and return one byte
}
```

In the example above, the `check()` function is used, not `require()`. These functions throw different exceptions:
`require()` throws an `IllegalArgumentException`, whereas `check()` throws an `IllegalStateException`. This difference 
might become significant when debugging.

## Avoid arrays in public signatures

Arrays are always mutable, and Kotlin is built around safe – read-only or immutable – objects. If you have to use arrays 
in your API, copy them before passing them anywhere so that you can check that they have not been modified. 
As an alternative, use read-only and mutable collections according to your intentions. Generally, it is best to avoid 
using arrays, and if you must, do so with extra caution.

For example, enum classes in Kotlin have the `values()` function that returns an array of all elements of the enum. 
If the array is not copied, a user is able to rewrite the elements:

```kotlin
enum class Test { A, B }

fun main() { Test.values()[0] = Test.B }
```

If you cache values inside the enum, the cache will be corrupted after running the code above. If the values are not 
cached, it's an additional runtime overhead for each call of the `values()` function.

This is the reason why Kotlin deprecated the `values()` functions since version 1.9 and introduced the `entries()`
function, that returns an immutable set.

## Avoid varargs

A `vararg` – [variable number of arguments](functions.md#variable-number-of-arguments-varargs) – works as an array 
under the hood, but the array elements are passed individually to the function, not the whole array. This operation is 
costly because it's copying the same array repeatedly.

Consider the following code:

```kotlin
fun printElements(delimiter: String, vararg elements: String) {
    for (i in elements.indices) {
        print(elements[i])
        if (i < elements.lastIndex) print(delimiter)
    }
}

fun printWithSpace(vararg elements: String) {
    printElements(" ", *elements)
}

fun main() {
    printWithSpace("x", "y", "z")
}
```
{kotlin-runnable="true" id ="jvm-api-guide-print-elements"}

The `printElements()` function prints all strings from the `vararg` argument `elements` with a delimiter, and 
the `printWithSpace()` function calls `printElements()` with the delimiter defined as a space. The code looks innocent: 
you just pass elements from `printWithSpace()` to `printElements()`. Without the spread operator `*`, the code won't compile, 
but with it, the **array is actually copied** before being passed to the `printElements()` function. The longer 
the chain is, the more copies are created and the bigger the unexpected memory overhead is.

## What's next?

Learn about APIs':
* [Debuggability](jvm-api-guidelines-debuggability.md)
* [Backward compatibility](jvm-api-guidelines-backward-compatibility.md)
