[//]: # (title: Minimizing mental complexity overview)

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

## Next step

To begin exploring these strategies in depth, you can start with learning about simplicity in the next section.

[Proceed to the next part](api-guidelines-simplicity.md)
