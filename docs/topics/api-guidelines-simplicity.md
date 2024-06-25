[//]: # (title: Simplicity)

The fewer concepts your users need to understand and the more explicitly these are communicated, the simpler their mental
model is likely to be. This can be achieved by limiting the number of operations and abstractions in the API.

Ensure that the [visibility](visibility-modifiers.md) of declarations in your library is set appropriately to keep internal implementation details
out of the public API. Only APIs that are explicitly designed and documented for public use should be accessible to users.

In the next part of the guide, we'll discuss some guidelines for promoting simplicity.

## Use explicit API mode

We recommend using the [explicit API mode](whatsnew14.md#explicit-api-mode-for-library-authors) feature of the Kotlin compiler,
which forces you to explicitly state your intentions when you're designing the API for your library.

With explicit API mode, you must:

* Add visibility modifiers to your declarations to make them public, instead of relying on the default public visibility. This ensures that you've considered what you're exposing as part of the public API.
* Define the types for all your public functions and properties to prevent unintended changes to your API from inferred types.

## Reuse existing concepts

One way to limit the size of your API is to reuse existing types. For example, instead of creating a new type for durations, you can use [`kotlin.time.Duration`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.time/-duration/).
This approach not only streamlines development but also improves interoperability with other libraries.

Be careful when relying on types from third-party libraries or platform-specific types, as they can tie your library to these elements.
In such cases, the costs may outweigh the benefits.

Reusing common types such as `String`, `Long`, `Pair`, and `Triple` can be effective, but this should not stop you from
developing abstract data types if they better encapsulate domain-specific logic.

## Define and build on top of core API

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

## Next step

In the next part of the guide, you'll learn about readability. 

[Proceed to the next part](api-guidelines-readability.md)
