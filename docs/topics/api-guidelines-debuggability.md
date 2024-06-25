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

## Next step

In the next part of the guide, you'll learn about testability.

[Proceed to the next part](api-guidelines-testability.md)
