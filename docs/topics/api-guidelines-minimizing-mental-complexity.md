[//]: # (title: Minimizing mental complexity)

Users need to quickly and accurately build a mental model of your library's functions and abstractions before using it. 
The best way to achieve this is by minimizing the amount of complexity they encounter.

Strategies for minimizing mental complexity include:

* **Simplicity:** Strive for an API that provides the most functionality with the fewest components, reusing existing Kotlin types and structures to avoid redundancy. Where possible, create a small set of core abstractions and build additional functionality on top of them.
* **Readability:** Write the API in a declarative style to make the code's intent clear. Choose names for abstractions directly from the problem domain, unless it's absolutely necessary to invent new ones. Use basic data types for their intended purposes. Clearly distinguish between core and optional functionality.
* **Consistency:** Maintain a single, clear approach for every design aspect of your API. Use uniform naming conventions, error handling strategies, and patterns, whether they are object-oriented or functional.
* **Predictability:** Design your library to adhere to [the ‘principle of least surprise'](https://en.wikipedia.org/wiki/Principle_of_least_astonishment). Ensure the default settings match the most common use cases, allowing users to accomplish their tasks with the simplest and shortest code. Allow extensions to your library only in clearly specified ways to maintain consistency and predictability.
* **Debuggability:** Ensure your library aids users in troubleshooting by facilitating the extraction of information and navigation through nested function calls. When exceptions are thrown, both the type and content of the exception should match the underlying issue, providing all necessary details to effectively diagnose and resolve problems. It should be possible to capture and output the state of domain objects, and to view any intermediate representations.
* **Testability:** Ensure that your library, as well as the code that uses it, can be easily tested.

The following sections give more detailed information on implementing these strategies in Kotlin.

## Simplicity

The fewer concepts your users need to understand and the more explicitly these are communicated, the simpler their mental 
model is likely to be. This can be achieved by limiting the number of operations and abstractions in the API.

Ensure that the [visibility](visibility-modifiers.md) of declarations in your library is set appropriately to keep internal implementation details 
out of the public API. Only APIs that are explicitly designed and documented for public use should be accessible to users.

In the next part of the guide, we'll discuss some guidelines for promoting simplicity.

### Use explicit API mode

We recommend using the [explicit API mode](whatsnew14.md#explicit-api-mode-for-library-authors) feature of the Kotlin compiler, which forces you to explicitly state your intentions when you're designing the API for your library.

With explicit API mode, you must:

* Add visibility modifiers to your declarations to make them public, instead of relying on the default public visibility. This ensures that you've considered what you're exposing as part of the public API.
* Define the types for all your public functions and properties to prevent unintended changes to your API from inferred types.

### Reuse existing concepts

One way to limit the size of your API is to reuse existing types. For example, instead of creating a new type for durations, you can use [`kotlin.time.Duration`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.time/-duration/). 
This approach not only streamlines development but also improves interoperability with other libraries.

Be careful when relying on types from third-party libraries or platform-specific types, as they can tie your library to these elements. 
In such cases, the costs may outweigh the benefits.

Reusing common types such as `String`, `Long`, `Pair`, and `Triple` can be effective, but this should not stop you from 
developing abstract data types if they better encapsulate domain-specific logic.

### Define and build on top of core API

Another route to simplicity is to define a small conceptual model based around a limited set of core operations. 
Once the behavior of these operations is clearly documented, you can expand the API by developing new operations that 
build directly on or combine these core functions.

For example:

* In the [Kotlin Flows API](flow.md), common operations like [`filter`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/filter.html) and [`map`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html) are built on top of the [`transform`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/transform.html) operation.
* In the [Kotlin Time API](time-measurement.md) the [`measureTime`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.time/measure-time.html) function utilizes [`TimeSource.Monotonic`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.time/-time-source/-monotonic/).

While it's often beneficial to base additional operations on these core components, it's not always necessary. 
You may find opportunities to introduce optimized or platform-specific variations that expand the functionality or adapt more broadly to different inputs.

As long as users are able to solve non-trivial problems with the core operations and can refactor their solutions with 
additional operations without altering any behavior, the simplicity of the conceptual model is preserved.


## Readability

Creating a readable API involves more than just writing clean code. 
It requires thoughtful design that simplifies integration and usage. 
This section explores how you can enhance API readability by structuring your library with composability in mind, 
utilizing domain-specific languages (DSLs) for concise and expressive setup, and using extension functions and properties 
for clear and maintainable code.

### Prefer Explicit Composability

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

### Use DSLs

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

### Use extension functions and properties

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

### Avoid using the boolean type as an argument

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

### Use numeric types appropriately

Kotlin defines a set of numeric types that you may use as part of your API. Here's how to use them appropriately:

* Use the `Int`,` Long` and `Double` types as arithmetic types. They represent values with which calculations are performed.
* Avoid using arithmetic types for non-arithmetic entities. For example, if you represent an ID as a `Long` your users
might be tempted to compare IDs, on the assumption they are assigned in order.
This could lead to unreliable or meaningless results, or create dependencies on implementations that could change without warning.
A better strategy is to define a specialized class for the ID abstraction. You could use [Inline value classes](inline-classes.md) to build such abstractions without affecting performance. See the  [`Duration`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.time/-duration/) class for an example.
* The `Byte`, `Float` and `Short` types are memory layout types. They are used to restrict the amount of memory available
for storing a value, such as in caches or when transmitting data over a network.
These types should only be used when the underlying data reliably fits within that type, and calculations are not required.
* The unsigned integer types `UByte`, `UShort`, `UInt` and `ULong` should be used to utilize the full range of positive
values available in a given format. They are suitable for scenarios requiring values beyond the range of signed types or
for interoperability with native libraries. However, avoid using them in situations where the domain only requires [non-negative integers](unsigned-integer-types.md#non-goals).

## Consistency

Consistency is crucial in API design to ensure ease of use. By maintaining consistent parameter order, naming conventions,
and error handling mechanisms, your library will be more intuitive and reliable for users. Following these best practices
helps avoid confusion and misuse, leading to a better developer experience and more robust applications.

### Preserve parameter order, naming and usage

When designing a library, maintain consistency in the ordering of arguments, the naming scheme, and the use of overloading.
For example, if one of your existing methods has `offset` and `length` parameters, you should not switch to alternatives like
`startIndex` and `endIndex` for a new method unless there is a compelling reason.

Overloaded functions provided by the library should behave identically.
Users expect the behavior to remain consistent when they change the type of a value they pass into your library.
For example, these calls all create identical instances, as the input is semantically the same:

```kotlin
BigDecimal(200)
BigDecimal(200L)
BigDecimal("200")
```

Avoid mixing parameter names like `startIndex` and `stopIndex` with synonyms like `beginIndex` and `endIndex`.
Similarly, choose one term for values in collections, such as `element`, `item`, `entry`, or `entity`, and stick with it.

Name related methods consistently and predictably. As an example, the Kotlin standard library contains pairs like
`first` and `firstOrNull`, `single` or `singleOrNull`.
These pairs clearly indicate that some might return `null` while others might throw an exception.
Parameters should be declared from the general to the specific, so essential inputs appear first and optional inputs last.
For example, in [`CharSequence.findAnyOf`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/find-any-of.html) the `strings` collection goes first, followed by the `startIndex`, and finally the `ignoreCase` flag.

Consider a library managing employee records, and provides the following API to search for employees:

```kotlin
fun findStaffBySeniority(
    startIndex: Int, 
    minYearsServiceExclusive: Int
): List<Employee>


fun findStaffByAge(
    minAgeInclusive: Int, 
    startIndex: Int
): List<Employee>
```

This API would be extremely hard to use correctly.
There are multiple parameters of the same type presented in an inconsistent order, and used in an inconsistent way.
Users of your library will likely make incorrect assumptions about new functions based on their experience with existing ones.

### Use Object-Oriented design for data and state

Kotlin supports both the Object-Oriented and Functional programming styles.
Use classes to represent data and state in your API. When the data and state is hierarchical, consider using inheritance.

If all the state required can be passed as parameters, prefer using top-level functions.
When calls to these functions will be chained, consider writing them as extension functions to improve readability.

### Choose the appropriate error handling mechanism

Kotlin provides several mechanisms for error handling.
Your API can throw an exception, return a `null` value, use a custom result type, or use the built-in [`Result`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-result/) type. Ensure that your library uses these options consistently and appropriately.

When data cannot be fetched or calculated, use a nullable return type and return `null` to indicate missing data.
In other cases, throw an exception or return a `Result` type.

Consider providing overloads of functions, where one throws an exception, while the other wraps it in a result type instead.
In these cases, use the `Catching` suffix to indicate that exceptions are caught in the function.
For example, the standard library has the [`run`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/run.html) and [`runCatching`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/run-catching.html) functions using this convention,
and the coroutines library has [`receive`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html) and [`receiveCatching`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive-catching.html) methods for channels.

Avoid using exceptions for normal control flow. Design your API to allow for condition checks before attempting operations, thus preventing unnecessary error handling.
[Command / Query Separation](https://martinfowler.com/bliki/CommandQuerySeparation.html) is a useful pattern that can be applied here.

### Maintain conventions and quality

The final aspect of consistency relates, not to the design of the library itself, but to maintaining a high level of quality.

You should use automated tools (linters) for static analysis to ensure your code follows both general Kotlin conventions
and project-specific conventions.

A Kotlin library should also provide a suite of unit and integration tests covering all documented behaviors of all the API entry points.
Tests should include a wide range of inputs, especially known boundary and edge cases. Any untested behavior should be assumed to be (at best) unreliable.

Use this suite of tests during development to verify that changes do not break existing behavior.
Run these tests on every release as part of a standardized build and release pipeline.
Tools like [Kover](https://github.com/Kotlin/kotlinx-kover) can be integrated into your build process to measure coverage and generate reports.

## Predictability

To design a robust and user-friendly Kotlin library, it's essential to anticipate common use cases, allow for extensibility, and enforce proper usage.
Following best practices for default settings, error handling, and state management ensures a seamless experience
for users while maintaining the integrity and quality of the library.

### Do the right thing by default

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

### Allow opportunities for extension

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
As a library author, you can make this easier by [designing with extensions](#use-extension-functions-and-properties) in mind, and ensuring your library's types have clear core concepts.

### Prevent unwanted and invalid extensions

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

### Avoid exposing mutable state

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

### Validate inputs and state

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

## Debuggability

Users of your library will build on its functionality, and the features they build will contain errors that need to be identified and resolved.
This error resolution process might be conducted within a debugger during development or using logging and observability tools in production.
Your library can follow these best practices to make debugging it easier.

### Provide a toString method for stateful types

For every type that contains state, provide a meaningful `toString` implementation.
This implementation should return an intelligible representation of the instance's current content, even for internal types.

Since `toString` representations of types are often written to logs, consider security when implementing this method and
avoid returning sensitive user data.

Ensure the format used to describe the state is as consistent as possible across the different types in your library.
This format should be explicitly described and thoroughly documented when it is part of a contract implemented by your API.
The output from your `toString` methods may support parsing, for example in automated test suites.

For example, consider the following types from a library supporting service subscriptions:

```kotlin
enum class SubscriptionResultReason {
    Success, InsufficientFunds, IncompatibleAccount
}


class SubscriptionResult(
    val result: Boolean,
    val reason: SubscriptionResultReason,
    val description: String
)
```

Without a `toString` method, printing a `SubscriptionResult` instance is not very useful:

```kotlin
fun main() {
    val result = SubscriptionResult(
       false,
       IncompatibleAccount,
       "Users account does not support this type of subscription"
    )




    //prints 'org.example.SubscriptionResult@13221655'
    println(result)
}
```

Nor is the information readily displayed in the debugger:

![Results in the debugger](debugger-result.png){width=500}

Adding a simple `toString` implementation improves the output significantly in both cases:

```kotlin
//prints 'Subscription failed (reason=IncompatibleAccount, description="Users 
// account does not support this type of subscription")'
override fun toString(): String {
    val resultText = if(result) "succeeded" else "failed"
    return "Subscription $resultText (reason=$reason, description=\"$description\")"
}
```

![Adding toString results in a much better result](debugger-result-tostring.png){width=700}

While it might be tempting to use data classes to gain a `toString` method automatically, it's not  recommended for backward compatibility reasons.
Data classes are discussed in more detail in the [Avoid using data classes in your API](api-guidelines-backward-compatibility.md#avoid-using-data-classes-in-your-api) section.

Note that the state described in the `toString` method does not need to be information from the problem domain.
It can relate to the status of ongoing requests (as in the example above), the health of connections to external services,
or intermediate state within an ongoing operation.

For example, consider the following builder type:

```kotlin
class Person(
    val name: String?,
    val age: Int?,
    val children: List<Person>
) {
    override fun toString(): String =
        "Person(name=$name, age=$age, children=$children)"
}

class PersonBuilder {
    var name: String? = null
    var age: Int? = null
    val children = arrayListOf<Person>()


    fun child(personBuilder: PersonBuilder.() -> Unit = {}) {
       children.add(person(personBuilder))
    }
    fun build(): Person = Person(name, age, children)
}


fun person(personBuilder: PersonBuilder.() -> Unit = {}): Person = 
    PersonBuilder().apply(personBuilder).build()
```

This is how you would use this type:

![Using the builder type example](halt-breakpoint.png){width=500}

If you halt the code at the breakpoint displayed on the image above, the information displayed will not be helpful:

![Halting code at the breakpoint result](halt-result.png){width=500}

Adding a simple `toString` implementation results in a much more helpful output:

```kotlin
override fun toString(): String =
    "PersonBuilder(name=$name, age=$age, children=$children)"
```

With this addition, the debugger shows:

![Adding toString to the halt point](halt-tostring-result.png){width=700}

This way, you can immediately see which fields are set and which are not.

### Adopt and document a policy for handling exceptions

As discussed in the [Choose appropriate error handling mechanism](#choose-the-appropriate-error-handling-mechanism) section,
there are occasions when it's appropriate for your library to throw an exception to signal an error.
You may create your own exception types for this purpose.

Libraries that abstract and simplify low-level APIs will also need to handle exceptions thrown by their dependencies.
A library might choose to suppress the exception, pass it on as it is, convert it to a different type of an exception,
or signal the error to users in a different way.

Any of these options could be valid, depending on the context. For example:

* If a user adopts library A purely for the convenience of simplifying library B, it may be appropriate for library A to rethrow any exceptions generated by library B without modification.
* If library A adopts library B purely as an internal implementation detail, then library-specific exceptions thrown by library B should never be exposed to users of library A.

You must adopt and document a consistent approach to exception handling so users can make productive use of your library.
This is especially important for debugging. Users of your library should be able to recognise, in the debugger and in logs,
when an exception has originated from your library.

The type of the exception should indicate the type of the error, and the data in the exception should help the user
locate the root cause of the issue.
A common pattern is to wrap a low-level exception in a library-specific one, with the original exception accessible as the `cause`.

## Testability

In addition to [testing your library](#maintain-conventions-and-quality), ensure that code using your library is also testable.

### Avoid global state and stateful top-level functions

Your library should not rely on state in global variables or provide stateful top-level functions as part of its public API.
Such variables and functions make testing code that uses the library difficult, as tests need to find ways to control these global values.

For example, a library might define a globally accessible function that provides access to the current time:

```kotlin
val instant: Instant = Clock.now()
println(instant)
```

Any code using this API will be difficult to test, as the call to the `now()` function will always return the real current time, while in tests it's often desirable to return fake values instead.

To enable testability, the [`kotlinx-datetime`](https://github.com/Kotlin/kotlinx-datetime) library has an API that lets
users get a `Clock` instance, and then use that to get the current time:

```kotlin
val clock: Clock = Clock.System
val instant: Instant = clock.now()
println(instant)
```

This allows users of a library to inject a `Clock` instance into their own classes, and replace the real implementation
with a fake one during tests.

## What's next

If you haven't already, consider checking out these pages:

* Learn about maintaining backward compatibility in the [Backward compatibility](api-guidelines-backward-compatibility.md) page.
* For an extensive overview on effective documentation practices, see [Informative Documentation](api-guidelines-informative-documentation.md).
