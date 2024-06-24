[//]: # (title: Predictability)

To design a robust and user-friendly Kotlin library, it's essential to anticipate common use cases, allow for extensibility, and enforce proper usage.
Following best practices for default settings, error handling, and state management ensures a seamless experience
for users while maintaining the integrity and quality of the library.

## Do the right thing by default

Your library should anticipate the "happy path" for each use case, and provide default settings accordingly.
Users should not need to supply default values for the library to function correctly.

For example, when using the [Ktor `HttpClient`](https://ktor.io/docs/client-create-new-application.html) the most common
use case is sending a GET request to the server.
This can be accomplished using the code below, where only essential information needs to be specified:

```kotlin
val client = HttpClient(CIO)
val response: HttpResponse = client.get("https://ktor.io/")
```

It's not necessary to provide values for mandatory HTTP headers or custom event handlers for possible status codes in the response.

If there is no obvious "happy path" for a use case or if a parameter should have a default value but there is no non-contentious option,
it likely indicates a flaw in the requirement analysis.

## Allow opportunities for extension

When the correct choice cannot be anticipated, allow users to specify their preferred approach.
Your library should also let the user supply their own approach or use a third-party extension.

For example, with the [Ktor `HttpClient`](https://ktor.io/docs/client-serialization.html), users are encouraged to install
support for content negotiation when configuring the client, and to specify their preferred serialization formats:

```kotlin
val client = HttpClient(CIO) {
    install(ContentNegotiation) {
        json(Json {
            prettyPrint = true
            isLenient = true
        })
    }
}
```

Users can choose which plugins to install or create their own using the [separate API for defining client plugins](https://ktor.io/docs/client-custom-plugins.html).

Additionally, users can define extension functions and properties for types in the library.
As a library author, you can make this easier by [designing with extensions](api-guidelines-readability.md#use-extension-functions-and-properties) in mind,
and ensuring your library's types have clear core concepts.

## Prevent unwanted and invalid extensions

Users should not be able to extend your library in ways that violate its original design or are impossible within the rules of the problem domain.

For example, when marshaling data to and from JSON, only six types are supported in the output format:
`object`, `array`, `number`, `string`, `boolean`, and `null`.

If you create an open class or interface called `JsonElement`, users could create invalid derived types, like `JsonDate`.
Instead, you can make the `JsonElement` interface sealed and provide an implementation for each type:

```kotlin
sealed interface JsonElement

class JsonNumber(val value: Number) : JsonElement
class JsonObject(val values: Map<String, JsonElement>) : JsonElement
class JsonArray(val values: List<JsonElement>) : JsonElement
class JsonBoolean(val value: Boolean) : JsonElement
class JsonString(val value: String) : JsonElement
object JsonNull : JsonElement
```

Sealed types also enable the compiler to ensure your `when` expressions are exhaustive, without requiring an `else` statement,
improving readability and consistency.

## Avoid exposing mutable state

When managing multiple values, your API should, whenever possible, accept and/or return read-only collections.
Mutable collections are not thread-safe and introduce complexity and unpredictability into your library.

For example, if a user modifies a mutable collection returned from an API entry point,
it will be unclear whether they are modifying the implementation's structure or a copy.
Similarly, if users can modify the values within a collection after passing it to a library, it will be unclear whether
this affects the implementation.

Since arrays are mutable collections, avoid using them in your API.
If arrays must be used, make defensive copies before sharing data with users. This ensures your data structures remain unmodified.

This policy of making defensive copies is automatically performed by the compiler for `vararg` arguments.
When using the spread operator to pass an existing array where a `vararg` argument is expected, a copy of your array is automatically created.

This behavior is demonstrated in the following example:

```kotlin
fun main() {
    fun demo(vararg input: String): Array<out String> = input

    val originalArray = arrayOf("one", "two", "three", "four")
    val newArray = demo(*originalArray)

    originalArray[1] = "ten"

    //prints "one, ten, three, four"
    println(originalArray.joinToString())

    //prints "one, two, three, four"
    println(newArray.joinToString())
}
```

## Validate inputs and state

Ensure that your library is used correctly by validating inputs and existing state before implementation proceeds.
Use the [`require`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/require.html) function to verify inputs and the [`check`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/check.html) function to validate existing state.

The `require` function throws an [`IllegalArgumentException`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-illegal-argument-exception/#kotlin.IllegalArgumentException) if its condition is `false`, causing the function to fail immediately with an appropriate error message:

```kotlin
fun saveUser(username: String, password: String) {
    require(username.isNotBlank()) { "Username should not be blank" }
    require(username.all { it.isLetterOrDigit() }) {
        "Username can only contain letters and digits, was: $username"
    }
    require(password.isNotBlank()) { "Password should not be blank" }
    require(password.length >= 7) {
        "Password must contain at least 7 characters"
    }

    /* Implementation can proceed */
}

```

Error messages should include relevant inputs to help users determine the cause of the failure, as shown above by the
error message for usernames that contain invalid characters, which includes the incorrect username.
An exception to this practice is when including a value in the error message could reveal information that might be used
maliciously as part of a security exploit, which is why the error message for the password's length does not include the password input.

Similarly, the `check` function throws an [`IllegalStateException`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-illegal-state-exception/#kotlin.IllegalStateException) if its condition is `false`.
Use this function to verify the state of an instance, as shown in the example below:

```kotlin
class ShoppingCart {
    private val contents = mutableListOf<Item>()

    fun addItem(item: Item) {
       contents.add(item)
    }

    fun purchase(): Amount {
       check(contents.isNotEmpty()) {
           "Cannot purchase an empty cart"
       }
       // Calculate and return amount
    }
}
```

## Next step

In the next part of the guide, you'll learn about debuggability.

[Proceed to the next part](api-guidelines-debuggability.md)
