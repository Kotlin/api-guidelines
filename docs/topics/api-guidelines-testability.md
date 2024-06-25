[//]: # (title: Testability)

In addition to [testing your library](api-guidelines-consistency.md#maintain-conventions-and-quality), ensure that code using your library is also testable.

## Avoid global state and stateful top-level functions

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
