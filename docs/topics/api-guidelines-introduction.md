[//]: # (title: Introduction to library authors' guidelines)

This guide contains a summary of best practices and ideas to consider when designing libraries.

To be effective, a library must achieve certain fundamental objectives. Specifically, it should:

* Define its problem domain and implement a set of related functional requirements that solve the problems it defined. 
For example, an HTTP client may aim to support all HTTP request types and understand the various headers, content types, and status codes.
* Meet the non-functional criteria appropriate to the problem domain. These typically involve performance, reliability, security, and usability. 
The relative importance of these criteria varies widely. For example, a library designed for batch processing may not require the same level of performance as one intended for day trading.

    > The process of identifying and defining functional and non-functional requirements is a complex topic that has been extensively studied in software engineering.
    > This guide does not cover these topics in depth, as they are beyond its scope.
    >
    {style="note"}

The main focus of this guide is to explore the characteristics that a library must have to stay relevant and popular with its users. These characteristics include:

* **Minimize mental complexity:**  All developers must consider the readability and maintainability of their code. It's crucial to reduce the mental effort required for others to read, understand, and use your APIs. Achieving this involves creating libraries that are clear, consistent, predictable, and easy to debug.
* **Backward compatibility:** When releasing a new version of an API, ensure the existing API remains operational. Clearly communicate and document any breaking changes well in advance. Provide a straightforward, clear, and gradual pathway for users to adopt the new API or design changes.
* **Informative documentation:** The documentation that accompanies the library needs to do more than repeat function and type declarations. It should be comprehensive and specifically tailored to the library's audience. It should accurately reflect the needs and scenarios of various user roles, ensuring it provides essential information without being overly simplistic or complex. Always include clear examples, balancing explanatory text with practical code samples.

Additionally, **building your Kotlin library with multiplatform support** can broaden its applicability across projects targeting various environments.
Designing APIs to work reliably in both shared and platform-specific code can improve the library’s versatility and usability across all supported targets.

The following sections dig deeper into these characteristics, with practical advice on how you can provide the 
best possible experience to users of your libraries.

## What's next

* Explore strategies for minimizing mental complexity in [Minimizing mental complexity](api-guidelines-minimizing-mental-complexity.md).
* Learn about maintaining backward compatibility in [Backward compatibility](api-guidelines-backward-compatibility.md).
* For an extensive overview on effective documentation practices, see [Informative documentation](api-guidelines-informative-documentation.md).
* Discover best practices for building multiplatform libraries in [Building Kotlin library for multiplatform](api-guidelines-build-for-multiplatform.md).