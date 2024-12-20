[//]: # (title: Building a Kotlin library for multiplatform)

When creating a Kotlin library, consider building and [publishing it with support for Kotlin Multiplatform](multiplatform-publish-lib.md).
This broadens the target audience of your library, making it compatible with projects targeting multiple platforms.

The following sections provide guidelines to help you build a Kotlin Multiplatform library effectively.

## Maximize your reach

To make your library available to the largest number of projects as a dependency,
aim to support as many [target platforms](multiplatform-dsl-reference.md#targets) of Kotlin Multiplatform as possible.

If your library doesn't support the platforms used by a multiplatform project,
whether it's a library or an application, it becomes difficult for that project to depend on your library.
In that case, projects can use your library for some platforms and need to implement separate solutions for others,
or they will choose an alternative library altogether that supports all their platforms.

To streamline artifact production, you can try Experimental [cross-compilation](multiplatform-publish-lib.md#host-requirements) to publish Kotlin Multiplatform libraries from any host.
This allows you to generate `.klib` artifacts for Apple targets without an Apple machine.
We plan to stabilize this feature and further improve library publication in the future.
Please leave your feedback about this feature in our issue tracker [YouTrack](https://youtrack.jetbrains.com/issue/KT-71290).

> For Kotlin/Native targets, consider using a [tiered approach](native-target-support.md#for-library-authors) to support all possible targets.
>
{style="note"}

## Design APIs for use from common code

When creating a library, design the APIs to be usable from common Kotlin code, instead of writing platform-specific implementations.

Provide reasonable default configurations when possible and include platform-specific configuration options.
Good defaults allow users to use the library's APIs from common Kotlin code, without the need to write platform-specific implementations to configure the library.

Place APIs in the broadest relevant source set using the following priority:

* **The `commonMain` source set:** APIs in the `commonMain` source set are available to all platforms the library supports. Aim to place most of your library's API here.
* **Intermediate source sets:** If some platforms don't support certain APIs, use [intermediate source sets](multiplatform-discover-project.md#intermediate-source-sets) to target specific platforms.
For example, you can create a `concurrent` source set for targets that support multi-threading or a `nonJvm` source set for all non-JVM targets.
* **Platform-specific source sets:** For platform-specific APIs, use source sets like `androidMain`.

> To learn more about the source sets of Kotlin Multiplatform projects, see [Hierarchical project structure](multiplatform-hierarchy.md).
>
{style="tip"}

## Ensure consistent behavior across platforms

To ensure your library behaves consistently on all supported platforms,
APIs in a multiplatform library should accept the same range of valid inputs, perform the same actions,
and return the same results across all platforms.
Similarly, the library should treat invalid inputs uniformly and report errors or throw exceptions consistently on all platforms.

Inconsistent behavior makes the library difficult to use and forces users to add conditional logic in common code to manage platform-specific differences.

You can use [`expect` and `actual` declarations](multiplatform-expect-actual.md) to declare functions in common code with
platform-specific implementations that have full access to the native APIs of each platform.
These implementations must also have the same behavior to ensure that they can be used reliably from common code.

When APIs behave consistently across platforms, they only need to be documented once in the `commonMain` source set.

> If platform differences are unavoidable, such as when one platform
> supports a broader set of inputs, minimize them where possible. For example, you might not want to limit one platform's functionality to match the others. In such cases, clearly document the specific differences.
>
> {style=”note”}

## Test on all platforms

Multiplatform libraries can have [multiplatform tests](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-run-tests.html) written in common code that runs on all platforms.
Executing this common test suite regularly on your supported platforms can ensure that the library behaves correctly and consistently.

Regularly testing Kotlin/Native targets across all published platforms can be challenging.
However, to ensure broader compatibility, consider publishing the library for all the targets it can support, using a [tiered method](native-target-support.md#for-library-authors) when testing compatibility.

Use the [`kotlin-test`](https://kotlinlang.org/api/latest/kotlin.test/) library to write tests in common code and execute them with platform-specific test runners.

## Consider non-Kotlin users

Kotlin Multiplatform offers interoperability with native APIs and languages across its supported target platforms.
When creating a Kotlin Multiplatform library, consider whether users might need to use your library's types and declarations
from languages other than Kotlin.

For example, if some types from your library will be exposed to Swift code through interoperability,
design those types to be easily accessible from Swift.
The [Kotlin-Swift interopedia](https://github.com/kotlin-hands-on/kotlin-swift-interopedia) provides useful insights into how Kotlin APIs appear when called from Swift.

## Promote your library

Your library can be featured on the [JetBrains' search platform](https://klibs.io/).
It's designed to make it easy to look for Kotlin Multiplatform libraries based on their target platforms.

Libraries that meet the criteria are added automatically. For more information on how to add your library, see [FAQ](https://klibs.io/faq).
