[//]: # (title: Consistency)

Consistency is crucial in API design to ensure ease of use. By maintaining consistent parameter order, naming conventions,
and error handling mechanisms, your library will be more intuitive and reliable for users. Following these best practices
helps avoid confusion and misuse, leading to a better developer experience and more robust applications.

## Preserve parameter order, naming and usage

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

## Use Object-Oriented design for data and state

Kotlin supports both the Object-Oriented and Functional programming styles.
Use classes to represent data and state in your API. When the data and state is hierarchical, consider using inheritance.

If all the state required can be passed as parameters, prefer using top-level functions.
When calls to these functions will be chained, consider writing them as extension functions to improve readability.

## Choose the appropriate error handling mechanism

Kotlin provides several mechanisms for error handling.
Your API can throw an exception, return a `null` value, use a custom result type, or use the built-in [`Result`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-result/) type.
Ensure that your library uses these options consistently and appropriately.

When data cannot be fetched or calculated, use a nullable return type and return `null` to indicate missing data.
In other cases, throw an exception or return a `Result` type.

Consider providing overloads of functions, where one throws an exception, while the other wraps it in a result type instead.
In these cases, use the `Catching` suffix to indicate that exceptions are caught in the function.
For example, the standard library has the [`run`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/run.html) and [`runCatching`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/run-catching.html) functions using this convention,
and the coroutines library has [`receive`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html) and [`receiveCatching`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive-catching.html) methods for channels.

Avoid using exceptions for normal control flow. Design your API to allow for condition checks before attempting operations,
thus preventing unnecessary error handling.
[Command / Query Separation](https://martinfowler.com/bliki/CommandQuerySeparation.html) is a useful pattern that can be applied here.

## Maintain conventions and quality

The final aspect of consistency relates, not to the design of the library itself, but to maintaining a high level of quality.

You should use automated tools (linters) for static analysis to ensure your code follows both general Kotlin conventions
and project-specific conventions.

A Kotlin library should also provide a suite of unit and integration tests covering all documented behaviors of all the API entry points.
Tests should include a wide range of inputs, especially known boundary and edge cases. Any untested behavior should be assumed to be (at best) unreliable.

Use this suite of tests during development to verify that changes do not break existing behavior.
Run these tests on every release as part of a standardized build and release pipeline.
Tools like [Kover](https://github.com/Kotlin/kotlinx-kover) can be integrated into your build process to measure coverage and generate reports.

## Next step

In the next part of the guide, you'll learn about predictability.

[Proceed to the next part](api-guidelines-predictability.md)
