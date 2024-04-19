
Guidelines for samples in documentation 
--------------------------------------------------------
The guideline provides a skeleton of what substitutes a good sample to the Kotlin API entry. These are general recommendations, not strict rules. If ignoring some of the recommendations improves your documentation -- do that.

### Benefits of the samples

Samples provide quick-to-grasp and easy-to-reuse snippets of code. Users will be reading, rewriting and using them as an idiomatic approach to solve their domain-specific problems.
Samples alone shape the way your users use your API.

A well-crafted sample provides an unreasonably high number of benefits that you can consider as success criteria for your sample:

* The sample immediately narrows a gap between conceptual understanding (e.g. "I want to generate a random number") and a practical application (e.g. "The code that generates a random number") without going through all the nuances
* The sample provides an alternative or more familiar view on the problems API is solving, replacing or simplifying further textual understanding
* The sample clearly demonstrates which problem the API solves and which it does not
* The sample serves as a baseline recipe for the common usage of the API and makes the overall documentation easier to maintain
* The sample provides an overview of other API capabilities and ties them together, providing a learning by example/navigation experience
* The sample is information-dense -- for straightforward things it replaces the KDoc explanation, and for less trivial problems, it complements the KDoc.

Our general suggestion is to supply each public non-trivial (i.e. not a constant) signature with a code sample.

### General recommendations

Samples shape the way your users use your API, so treat them as any other code in your codebase:

* Use the preferred in your project code style
* Lean towards idiomatic usage of your API
* Avoid using patterns that you won't use in a regular development or in tests -- ignored exceptions, unnamed boolean arguments and `!!` unless explicitly pointed
* Explain with inline comments what actions are performed or in what state objects are
* Use relevant APIs in your sample where applicable in the same way you use references in KDoc
* Provide the context of usage if necessary
* Focus on **why** these API can be used
*  **Team review explicit point: asserts vs stdlib conditions in samples**
    * I.e. `assertEquals(42, sampleOutput)` vs `check(sampleOutput == 42)`
    * Pros of asserts: explicitly asserting, very familiar, test-friendly (`@sample` is a test)
    * Cons: when copy-pasted as is, the code is unresolbed, `main` rarely has asserts behaviour
        * Seva: still can be ok as readability here trumps reusability?
        * Also, can try to push converter to idea -- asserts to the main code -> checks
    * Pros of `check`s: works out of the box, copy-pasteable, will work in scratches with a less implementation
    * Cons: might be slightly less readable, as checks are less familiar and also do not encapsulate the intent (e.g. `check(sample.contains("foo"))` vs `assertContains(sample, "foo")`
* Both `inline` code blocks for [context-heavy examples](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/callback-flow.html) and `@sample` tag can be used.
* It might be reasonable to provide multiple samples for a single signature -- starting from the most generic one and going down on specifics (interface implementation, error handling, pitfalls workarounds)

## Specific recommendations
### Functions returning a result

A function returning a result must be accompanied by an example that clearly
links every part of the input to the output.

It must also be clear which part of the input *didn't* meaningfully affect the
behavior of the function and was treated completely regularly.

For simple cases where the string representation of the output explains the
behavior of the function unambiguously, it's enough to include that:

```kotlin
" :) ".repeat(3) // " :)  :)  :) "
```

Here, it's clear that the string ` :) ` is repeated `3` times.
We also see that `repeat` doesn't strip any whitespaces, doesn't merge
neighboring characters, and generally, simply repeats the string.

#### Explaining return value

If the string representation is enough to explain the behavior of the
function, but there are several forms of the result, each one must be included.

```kotlin
"NonEmpty".ifEmpty { "fallback" } // "NonEmpty"
" ".ifEmpty { "fallback" } // " "
"".ifEmpty { "fallback" } // "fallback"
```

The middle example is included to show that a string of spaces is not
considered empty; that is, space is treated regularly.

```kotlin
val list = mutableListOf(1, 5, 7, 10)
list.binarySearch(5) // 1, the position where exactly 5 is located
list.binarySearch(8) // -4 == -(3 + 1), since 3 is the position where 8 should be located in inserted
```

#### Explaining function configuration

if the function behaviour is configurable, it should be explicitly reflected in the sample.
```kotlin
val string = "KOTLIN"
string.contentEquals("kotlin") // false
string.contentEquals("KOTLIN") // true

string.contentEquals("kotlin", ignoreCase = true) // true
```

It is immediately clear that the function behaviour can be additionally configured and how it affects the output.

#### Returning an object

If the output is an object, the sample must include enough usages of the object
to demonstrate the effect of each argument passed to the function.

```kotlin
val myFlow = flowOf(1, 2, 3)
myFlow.collect {
  println(it)
} // prints 1, then 2, then 3
```

### Functions performing an action

The action performed by an effectful function must be accompanied by a
"before" vs "after" comparison, where it should be clear what happened.

```kotlin
val list = mutableListOf(4, 4, 1, 3, 2, 5, 0)
// list == [4, 4, 1, 3, 2, 5, 0]
list.sort()
// list == [0, 1, 2, 3, 4, 4, 5]
```

We need duplicated elements to demonstrate that they are not merged into one.

```kotlin
val list = mutableListOf(4, 4, 1, 3, 2, 5, 0)
// list == [4, 4, 1, 3, 2, 5, 0]
list.removeAt(0) // 4
// list == [4, 1, 3, 2, 5, 0], as the first element was removed.

try {
    list.removeAt(100)
} catch (e: IndexOutOfBoundsException) {
    // the list doesn't have 101 elements
}
// TODO: decide on it depending on whether we use asserts or checks
```

Here, we need to duplicate the removed element to show that there is no special
behavior related to the value of the first element (that is, the other elements
with the same value don't get removed).

We also need `0` in the list to show that the `0` passed as the argument is
unrelated to it.

### Overrides of functions

An override of a function must have its own usage sample if the types are
different: either the covariant types are narrower or the contravariant types
are wider.

```kotlin
val myFlow = MutableStateFlow(0)
myFlow.collect {
  // process the input
} // can only exit with an exception
throw IllegalStateException("This code will never be executed")
```

### Classes

A class consists of three parts:

* How its objects are obtained;
* How its objects are used;
* What other entities are available in its namespace (objects, functions, etc.).

It is recommended providing top-level overview in the documentation for each of the aspect, ideally with the help of inline examples

#### Construction

The examples for constructors should be provided in the top-level documentation
for the class and follow the same principles as the documentation of functions.

```kotlin
val parentJob = Job()
// parentJob.children.toList() == listOf<Job>()
val childJob = Job(parentJob)
// parentJob.children.toList() == listOf<Job>(childJob)
// childJob.parent == parentJob
```

Here, we must demonstrate both the known properties of the output and the
mutating behavior of the constructor.

#### Usage

According to the guidelines, extension functions should typically encode the
"extra" behavior, whereas member functions encode the core behavior.

The top-level documentation should include an overview of the core behavior
(typically, of the member functions), with simplified examples of each to
quickly show one what behaviors are available. In these brief overviews,
the rules for providing samples to functions don't need to be followed;
instead, the sample must only demonstrate how *some* inputs and *some* forms of
the output are linked. Edge cases and uncommon paths can be ignored.
However, the rough idea of what the function does must still be deducible.

#### Entities in the same namespace

These don't need to be mentioned in the top-level documentation, except when
they are used for object construction or are encountered in common usage,
in which case they should be part of the corresponding samples.

The sample in the documentation of constructing a `TimeZone` should mention
`availableZoneIds`, for example:

```kotlin
TimeZone.of("Europe/Berlin")
for (id in TimeZone.availableZoneIds()) {
    TimeZone.of(id) // doesn't fail
}
```

Likewise, `Channel`'s construction sample should mention the constants in the
companion object:

```kotlin
val channel = Channel<Int>(capacity = Channel.CONFLATED)
// channel contains nothing
channel.trySend(1)
// channel contains `1`
channel.trySend(2)
// channel contains `2`
```

### Implementation and inheritance

If a class or an interface is open for an external implementation, consider providing an additional (potentially advanced) example on how to implement it and what is the rationale of the implementation. If any invariants should preserved, consider explaining them as well:

```kotlin
/**  
 * A [clock][Clock] that always returns a given fixed [instant].  
 * * Such a clock is recommended to be used in unit tests to fix the current moment of time  
 * for the sake of testability and to ensure the component does not depend on a system clock. 
 */
public fun Clock.Companion.fixed(instant: Instant): Clock = FixedClock(instant)  
  
private class FixedClock(private val instant: Instant) : Clock {  
    // We are allowed to do so -- now contract explicitly allows this behaviour  
    override fun now(): Instant = instant  
}
```