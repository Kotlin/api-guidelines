[//]: # (title: Backward compatibility guidelines for library authors)

The most common motivation for creating a library is to expose functionality to a wider community.
This community might be a single team, a company, a particular industry, or a technology platform.
In every case backward compatibility will be an important consideration.
The wider the community the more important backward compatibility becomes, since you will be less aware of who your users
are and what constraints they work within.

Backward compatibility is not a single term, but can be defined at the binary, source and behavioral levels.
More information about these types is provided in this section.

Note that:

* It is possible to break binary compatibility without breaking source compatibility, as well as the other way around.
* It is desirable but very difficult to guarantee source compatibility. As a library author, you must consider every 
possible way a function or type could be invoked or instantiated by a user of the library.
Source compatibility is typically an aspiration, not a promise.

The rest of this section describes actions you can take, and tools you can use to help ensure the different kinds of compatibility.

## Compatibility types {initial-collapse-state="collapsed" collapsible="true"}

**Binary compatibility** means that a new version of a library can replace a previously compiled version of the library.
Any software that was compiled against the previous version of the library should continue to work correctly.

> Learn more about binary compatibility in the [Binary compatibility validator's README](https://github.com/Kotlin/binary-compatibility-validator?tab=readme-ov-file#what-makes-an-incompatible-change-to-the-public-binary-api) or from the [Evolving Java-based APIs](https://github.com/eclipse-platform/eclipse.platform/blob/master/docs/Evolving-Java-based-APIs-2.md) document.
>
{style="tip"}

**Source compatibility** means that a new version of a library can replace a previous one without modifying any of the
source code that uses the library. However, the outputs from compiling this client code may no longer
be compatible with the outputs from compiling the library, so the client code must be rebuilt against the new version of
the library to guarantee compatibility.

**Behavioral compatibility** means that a new version of the library does not modify the existing functionality,
except to fix bugs. The same features are involved and they have the same semantics.

## Use the Binary compatibility validator

JetBrains provides a [Binary compatibility validator](https://github.com/Kotlin/binary-compatibility-validator) tool, which can be used to ensure binary compatibility across different versions of your API.

This tool is implemented as a Gradle plugin, and it adds two tasks to your build:

* The `apiDump` task creates a human-readable `.api` file that describes your API.
* The `apiCheck` task compares the saved description of the API to classes compiled in the current build.

The `apiCheck` task is invoked at build time by the standard Gradle `check` task.
When compatibility is broken, the build fails. At that point, you should run the `apiDump` task manually and compare
the differences between the old and new versions.
If you are satisfied with the changes, you can update the existing `.api` file, which resides within your VCS.

The validator has [experimental support for validating KLibs](https://github.com/Kotlin/binary-compatibility-validator?tab=readme-ov-file#experimental-klib-abi-validation-support) produced by multiplatform libraries.

### Binary compatibility validation in the Kotlin Gradle plugin

<primary-label ref="experimental-general"/>

Starting with version 2.2.0, the Kotlin Gradle plugin supports binary compatibility validation. For more information,
see [Binary compatibility validation in the Kotlin Gradle plugin](gradle-binary-compatibility-validation.md).

## Specify return types explicitly

As discussed in the [Kotlin coding guidelines](coding-conventions.md#coding-conventions-for-libraries),
you should always explicitly specify function return types and property types within the API. See also the section about [Explicit API mode](api-guidelines-simplicity.md#use-explicit-api-mode).

Consider the following example, where the library author creates a `JsonDeserializer` and, for convenience, uses an extension function to associate it with the `Int` type:

```kotlin
class JsonDeserializer<T>(private val fromJson: (String) -> T) {
    fun deserialize(input: String): T {
        ...
    }
}

fun Int.defaultDeserializer() = JsonDeserializer { ... }
```

Let's say the author replaces this implementation with a `JsonOrXmlDeserializer`:

```kotlin
class JsonOrXmlDeserializer<T>(
    private val fromJson: (String) -> T,
    private val fromXML: (String) -> T
) {
    fun deserialize(input: String): T {
        ...
    }
}

fun Int.defaultDeserializer() = JsonOrXmlDeserializer({ ... }, { ... })
```

Existing functionality will continue to work, with the added ability to deserialize XML. However, this breaks binary compatibility.

## Avoid adding arguments to existing API functions

Adding non-default arguments to a public API breaks both binary and source compatibility,
as users are required to provide more information on an invocation than before.
However, even adding [default arguments](functions.md#parameters-with-default-values) can break compatibility.

For example, imagine you have the following function in `lib.kt`:

```kotlin
fun fib() = … // Returns zero
```

And the following function in `client.kt`:

```kotlin
fun main() {
    println(fib()) // Prints zero
}
```
Compiling these two files on the JVM would produce the outputs `LibKt.class` and `ClientKt.class`.

Let's say you reimplement and compile the `fib` function to represent the Fibonacci sequence, such that `fib(3)` returns 2,
`fib(4)` returns 3, and so on.
You add a parameter but give it a default value of zero to preserve the existing behavior:

```kotlin
fun fib(input: Int = 0) = … // Returns Fibonacci member
```

You now need to recompile the file `lib.kt`. You might expect that the `client.kt` file does not need to be recompiled,
and the associated class file can be invoked as follows:

```shell
$ kotlin ClientKt.class
```

But if you try this, a `NoSuchMethodError` occurs:

```text
Exception in thread "main" java.lang.NoSuchMethodError: 'int LibKt.fib()'
       at LibKt.main(fib.kt:2)
       at LibKt.main(fib.kt)
       …
```

This is because the signature of the method changed in the bytecode generated by the Kotlin/JVM compiler, breaking binary compatibility.

Source compatibility, however, is preserved. If you recompile both files, the program will run as before.

### Use manual overloads to preserve compatibility {initial-collapse-state="collapsed" collapsible="true"}

For all Kotlin targets, you must manually create several overloads of your function instead of a single function
that accepts default arguments to preserve binary compatibility. In the example above, this means creating a separate 
`fib` function for the case where you wish to take an `Int` parameter:

```kotlin
fun fib() = … 
fun fib(input: Int) = …
```

Use caution when using [`@JvmOverloads`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/-jvm-overloads/) to create overloads
for binary compatibility. 
Even though the annotation generates one overload of the function per default parameter, when Kotlin clients call the
function without including all the parameters then the compiled code will use a synthetitc `$default` version. When a new default parameter
is added to the function the synthetic version of the function will change; this will break binary compatitbility for code that compiled against
older versions of the library.

## Avoid widening or narrowing return types

When evolving an API, it is common to want to widen or narrow the return type of a function.
For example, in an upcoming version of your API, you might want to switch a return type from `List` to `Collection`
or from `Collection` to `List`.

You might want to narrow the type to `List` to meet user requests for indexing support.
Conversely, you might want to widen the type to `Collection` because you realize the data you are working with has no natural order.

It is easy to see why widening a return type breaks compatibility. For example, converting from `List` to `Collection`
breaks all the code that uses indexing.

You might think that narrowing a return type, for example from `Collection` to `List` would preserve compatibility.
Unfortunately, while source compatibility is preserved, binary compatibility is broken.

Let's say you have a demo function in the file `Library.kt`:

```kotlin
public fun demo(): Number = 3
```

And a client for the function in `Client.kt`:

```kotlin
fun main() {
    println(demo()) // Prints 3
}
```

Let's imagine a scenario where you change the return type of demo and only recompile `Library.kt`:

```kotlin
fun demo(): Int = 3
```

When you rerun the client, the following error will occur (on the JVM):

```text
Exception in thread "main" java.lang.NoSuchMethodError: 'java.lang.Number Library.demo()'
        at ClientKt.main(call.kt:2)
        at ClientKt.main(call.kt)
        …
```

This happens because of the following instruction in the bytecode generated from the `main` method:

```text
0: invokestatic  #12 // Method Library.demo:()Ljava/lang/Number;
```

The JVM is trying to invoke a static method called demo which returns a `Number`.
However, as this method no longer exists, you have broken binary compatibility.

## Avoid using data classes in your API

In regular development, the strength of data classes is the extra functions that are generated for you.
In API design, this strength becomes a weakness.

For example, let's say you use the following data class in your API:

```kotlin
data class User(
    val name: String,
    val email: String
)
```

Later, you might want to add a property called `active`:

```kotlin
data class User(
    val name: String,
    val email: String,
    val active: Boolean = true
)
```

This would break binary compatibility in two ways. Firstly, the generated constructor will have a different signature.
Additionally, the signature of the generated `copy` method changes.

The original signature (on Kotlin/JVM) would be:

```text
public final User copy(java.lang.String, java.lang.String)
```

After adding the `active` property, the signature becomes:

```text
public final User copy(java.lang.String, java.lang.String, boolean)
```

As with the constructor, this breaks binary compatibility.

It's possible to work around these issues by manually writing a secondary constructor and overriding the `copy` method.
However, the effort involved negates the convenience of using a data class.

Another issue with data classes is that changing the order of constructor arguments affects the generated `componentX` methods,
which are used for destructuring. Even if it does not break binary compatibility, changing the order will definitely break behavioral compatibility.

## Considerations for using the PublishedApi annotation

Kotlin allows inline functions to be a part of your library's API. Calls to these functions will be inlined into the
client code written by your users. This can introduce compatibility issues, so these functions are not allowed to call non-public-API declarations.

If you need to call an internal API of your library from an inlined public function, you can do so by annotating it with [`@PublishedApi`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-published-api/).
This makes the internal declaration effectively public, as references to it will end up in compiled client code.
Therefore, it must be treated the same as public declarations when making changes to it, as these changes might affect binary compatibility.

## Evolve APIs pragmatically

There are cases where you need to make breaking changes to your library's API over time by removing or changing an existing declaration.
In this section, we'll discuss how to handle such cases pragmatically.

When users upgrade to a newer version of your library, they should not end up with unresolved references to your library's
APIs in their project's source code. Instead of immediately removing something from your library's public API, you should follow a deprecation cycle. This way, you give your users time to migrate to an alternative.

Use the [`@Deprecated`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-deprecated/) annotation on the old declaration to indicate that it's being replaced. The parameters of this annotation provide important details about the deprecation:

* The `message` should explain what's being changed and why.
* The `replaceWith` parameter should be used where possible to provide automatic migration to a new API.
* The deprecation's level should be used to deprecate the API gradually. For more information, see the [Deprecated page of the Kotlin documentation](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-deprecated/).

Generally, a deprecation should first produce a warning, then an error, and then hide the declaration.
This process should occur across several minor releases, giving users time to make any required changes in their projects.
Breaking changes, such as removing an API, should happen only in major releases.
A library may adopt different versioning and deprecation strategies, but this must be communicated to its users to set the correct expectations.

You can learn more in the [Kotlin Evolution principles document](kotlin-evolution-principles.md#libraries) or in the [Evolving your Kotlin API painlessly for clients talk](https://www.youtube.com/watch?v=cCgXtpVPO-o&t=1468s)
by Leonid Startsev from KotlinConf 2023.

## Use the RequiresOptIn mechanism 

The Kotlin standard library [provides the opt-in mechanism](opt-in-requirements.md) to require
explicit consent from users before they use a part of your API.
This is based on creating marker annotations, which are themselves annotated with [`@RequiresOptIn`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-requires-opt-in/).
You should use this mechanism to manage expectations concerning source and behavioral compatibility, especially
when introducing new APIs to your library.

If you choose to use this mechanism, we recommend following these best practices:

* Use the opt-in mechanism to provide different guarantees to different parts of the API. For example, you could mark features as _Preview_, _Experimental_, and _Delicate_. Each category should be clearly explained in your documentation and in [KDoc comments](kotlin-doc.md), with appropriate warning messages.
* If your library uses an experimental API, [propagate the annotation](opt-in-requirements.md#propagate-opt-in-requirements) to your own users. This ensures your users are aware that you have dependencies which are still evolving.
* Avoid using the opt-in mechanism to deprecate already existing declarations in your library. Use `@Deprecated` instead, as described in the [Evolve APIs pragmatically](#evolve-apis-pragmatically) section.

## What's next

If you haven't already, consider checking out these pages:

* Explore strategies for minimizing mental complexity in the [Minimizing mental complexity](api-guidelines-minimizing-mental-complexity.md) page.
* For an extensive overview on effective documentation practices, see [Informative documentation](api-guidelines-informative-documentation.md).
