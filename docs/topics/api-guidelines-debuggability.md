[//]: # (title: Debuggability)

Users of your library will build on its functionality, and the features they build will contain errors that need to be identified and resolved.
This error resolution process might be conducted within a debugger during development or using logging and observability tools in production.
Your library can follow these best practices to make debugging it easier.

## Provide a toString method for stateful types

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

## Adopt and document a policy for handling exceptions

As discussed in the [Choose appropriate error handling mechanism](api-guidelines-consistency.md#choose-the-appropriate-error-handling-mechanism) section,
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

## Support coroutine stack trace recovery for custom exceptions
<primary-label ref="experimental-general"/>

You can add support for coroutine [stack trace recovery](coroutines-debugging.md#stack-trace-recovery) to custom exception types in your library to make them easier to debug.
This improves your library's support for the `kotlinx.coroutines` library and other asynchronous runtimes in Kotlin.

When a coroutine receives an exception from another coroutine through a suspending function, stack trace recovery creates a copy of the exception with the stack frames that lead to that function call.

The `kotlinx.coroutines` library performs stack trace recovery automatically for exceptions with constructors that take only an exception message, a cause, both, or no arguments.
If an exception type in your library requires additional constructor arguments, such as a line number or an error code, implement the [`StackTraceRecoverable`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.coroutines.debug/-stack-trace-recoverable/) interface.

To implement the interface, override the [`copyForStackTraceRecovery()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.coroutines.debug/-stack-trace-recoverable/copy-for-stack-trace-recovery.html) function.
In the override, return a new exception instance for stack trace recovery, or `null` if you don't want the `kotlinx.coroutines` library to copy the exception.

The `StackTraceRecoverable` interface is part of the Kotlin standard library, so implementing it doesn't add a dependency on the `kotlinx.coroutines` library.

Here's an example of a custom exception that preserves a `line` property when it creates a new instance for stack trace recovery:

```kotlin
import kotlin.coroutines.ExperimentalStdlibCoroutineSupportApi
import kotlin.coroutines.debug.StackTraceRecoverable

@OptIn(ExperimentalStdlibCoroutineSupportApi::class)
class FileEditException
// The implementation requires a private constructor
// to pass the cause to the IllegalStateException constructor
private constructor(
    val line: Int,
    private val detail: String,
    cause: Throwable?,
) : IllegalStateException("When editing line $line: $detail", cause),
    // Implements StackTraceRecoverable for stack trace recovery
    StackTraceRecoverable<FileEditException> {

    constructor(line: Int, detail: String) : this(line, detail, null)

    // Copies the line number and message details
    override fun copyForStackTraceRecovery(): FileEditException =
        FileEditException(line, detail, this)
    }

fun main() {
    val original = FileEditException(15, "Unexpected token")
    
    // Normally, you don't need to call this function directly unless you're testing its behavior
    // The kotlinx.coroutines library invokes it automatically during stack trace recovery
    val copy = original.copyForStackTraceRecovery()

    println(copy.message)
    // When editing line 15: Unexpected token

    println(copy.cause == original)
    // true
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="2.4.20"}

## Next step

In the next part of the guide, you'll learn about testability.

[Proceed to the next part](api-guidelines-testability.md)
