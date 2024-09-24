[//]: # (title: Best practices for library authors to create informative documentation)

The documentation you provide for your library is crucial.
It can determine whether users investigate your library, adopt it within their projects, and persevere when they encounter difficulties.
Developers today have unprecedented choice between languages, libraries, frameworks and platforms.
Therefore, it is essential to engage and inform your users; otherwise they may pursue other options.

In the earliest versions of your library, feedback from users will be scarce.
Fortunately, creating and refining documentation can serve as a feedback loop to greatly increase the quality of your project.
As such, creating documentation should never be seen as a burden, and should not be pushed down the list of priorities when creating a library.

Effective documentation not only informs users but also drives the development and refinement of your library.
Here are several key ways documentation can guide your development process:

* You should be able to explain, in a couple of paragraphs, what your library does, who will benefit from using it, and what the advantages are over alternative approaches. If you cannot do this, reconsider the scope and objectives of your project.
* You should be able to create a "Getting Started" guide that can get a potential user up and running as quickly as possible. What counts as *quickly* will depend on the problem domain, but you can compare against similar libraries on other platforms. The guide should hook the user into a feedback loop that keeps getting easier and faster while always producing reliable results. Creating this guide will help you identify sudden increases in complexity (cliff edges) that could hinder the user's progress.
* The act of documenting a function forces you to consider all the edge cases, such as valid ranges of inputs, exceptions that might be thrown, and how performance degrades as the work increases. This can often lead to improvements in the function signatures and the underlying implementation.
* If the code required to initialize your library always eclipses the code required to accomplish a task, rethink your configuration options.
* If you cannot create clear examples of performing basic tasks with standard options, consider optimizing your API for day-to-day use.
* If you cannot demonstrate how to test your library without using real data sources and online services, consider providing test doubles for components that access the network (and the outside world in general).

The sooner you provide documentation for your library, the sooner it can be tested by real-world users.
Feedback from these tests can then be used to improve the design.

## Provide comprehensive documentation

Your library must provide sufficient documentation to let users adopt it with minimum effort.
This documentation should include:

* A Getting Started guide
* An in-depth description of the API
* Longer examples for common use cases (also known as recipes)
* Links to resources such as blogs, articles, webinars, and conference talks

The Getting Started guide should cover how your library integrates with the supported build systems.
It should include a brief description of the most commonly used entities, with small examples of how they are used.
At each point where the library interacts with the outside world, specify the necessary steps to configure the environment
and how to verify that these steps have been completed successfully.
State explicitly if no steps are necessary.

If possible, provide separate versions of the documentation for each version of the library you support.
This prevents users from viewing information that is either outdated or too recent.
When this is not possible, clearly mark sections of the documentation that relate to parts of the API that have been redesigned.

## Create personas for your users

Creating and evaluating documentation without a clear understanding of the intended audience is challenging.
Defining multiple personas for the types of users who will read your documentation can be helpful.

Consider the user's constraints, such as the pre-existing software stack within which they need to operate.
Reviewers of the documentation can adopt these personas to make their conclusions more meaningful.

When you lack specific information about your users, it's best to be pessimistic.
For example, do not assume expertise in the latest or most advanced features of Kotlin.
Keep your code examples as simple as possible.

Personas are especially useful when real-world users cannot be consulted due to time, budgetary constraints,
or confidentiality agreements.
Over time, as you gain a better understanding of your users, refine the personas to match their needs more accurately.

## Document by example whenever possible

Documentation by example is one of the most cost-effective ways to explain basic concepts to users.
Whenever possible, provide simple and clear code examples that help to explain or demonstrate the current topic or concept being discussed.

The KDoc documentation format lets you use [inline markup using Markdown](kotlin-doc.md#inline-markup) in your documentation comments.
Use inline code snippets in comments to showcase the usage of an API.
For an example, see the [source code](https://github.com/Kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-test/common/src/TestCoroutineDispatchers.kt) and [rendered documentation](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-unconfined-test-dispatcher.html) of the test dispatchers of the coroutines library.

Providing examples like this can avoid the need to write lengthy descriptions of expected inputs, possible outputs,
and failure modes.
However, the context for each example needs to be clear, as do the circumstances in which it is relevant.
Simply providing a folder of uncommented sample programs does not qualify as documentation.

## Thoroughly document your API

Every supported API entry point should be documented using [KDoc](kotlin-doc.md).

Kotlin's documentation engine, [Dokka](dokka-introduction.md), includes only public declarations in its outputs by default. As discussed in the [Simplicity](api-guidelines-simplicity.md) section,
you should minimize your public API and remove public entry points that you don't want users to access.
If there are APIs you cannot hide from users by controlling their visibility, omit them from the documentation using the [suppress directive](kotlin-doc.md#suppress).

Begin the description of entry points with a clear, high-level description of what the function does.
Avoid simply restating the signature in natural language.

For example, don't say “takes a `String` and returns a `Connection`” but instead perhaps
“Attempts to connect to the database specified by the input string, returning a `Connection` if successful, and throwing a `ConnectionTimeoutException` otherwise”.

Specify each input's expected values and behavior with different inputs.
Explain the range of valid values and what happens when invalid values are provided.
For example, if a string input is supposed to be a URL, describe what happens when the string is empty, invalid, uses an unsupported protocol,
or refers to a location that does not exist.

Document every exception an API entry point may throw. Discuss failure conditions in the general description and reserve the exception section for detailed information. This enhances readability and helps the reader focus. Instead, include this information organically into the general description. Providing usage examples whenever possible also helps users understand how to implement the API correctly.

> We recommend learning about technical writing to improve the clarity and effectiveness of your documentation. 
> Consider exploring resources, such as this course from Google ([part one](https://developers.google.com/tech-writing/one) and [part two](https://developers.google.com/tech-writing/two)).
> 
{style="tip"}

## Document lambda parameters

When an API entry point takes a lambda, the user is providing some functionality that your library will execute on their behalf.
This requires extra documentation in at least two areas.

First, document what happens if the lambda throws an exception. Consider addressing the following questions:

* Will this cause an immediate failure, will the lambda be invoked repeatedly, or is there a fallback behavior?
* If the calling function needs to exit, will it rethrow the exception thrown from the lambda or a different one?
* If the exception is different, will it include the original?

Additionally, unless the function is declared as inline,document any special behavior relating to concurrency.
Ensure the following are covered:

* Will the lambda be invoked in the same thread as the caller?
* If the lambda is not invoked in the same thread as the caller, which thread (or thread pool) will it be invoked in?
* Could multiple copies of the lambda be run in parallel?
* Which other jobs of work may be using that thread?
* Can the user specify a thread for the library to use?
* Where multiple lambdas are being invoked, what guarantees are provided regarding sequencing?

## Use explicit links in documentation

It is very rare for an API entry point to be completely independent of other functionality in your library.
Typically, calls must be made in a specific sequence, there are multiple options for performing a particular task,
and entry points that perform related tasks are used in similar ways.
For example, functions like `format` and `parse` mirror each other.

Use the [`@see`](kotlin-doc.md#see-identifier) tag or [internal links](kotlin-doc.md#links-to-elements) to make these relationships explicit in your documentation.
This helps the reader by enabling them to [chunk](https://en.wikipedia.org/wiki/Chunking_(psychology)) the information together, building a better-integrated mental map of the library.

## Be self-contained where possible

When describing what makes an input valid, it is tempting to simply refer to the relevant standard, such as those produced
by the W3C, IEEE, or Unicode Consortium.
While providing these links can be helpful, the reader should not be forced to refer to an external specification to discover basic information,
like the set of whitespace characters.

Wherever possible, the documentation should be self-contained.
It should provide enough information for the user to understand the typical use of each API entry point.
You can refer the user to external documents for edge cases that will not happen in common use.

## Use simple English

It's important to use simple and clear English when creating documentation.
This ensures that your content is accessible to a global audience, including those whose first language is not English.
Avoid using complex words, jargon, Latin phrases, or idiomatic expressions that might confuse readers.
Instead, use straightforward language and concise sentences.

Simple English also makes the documentation easier to translate if needed.
Clear, unambiguous text reduces the risk of misinterpretation and improves the overall readability. 

## What's next

If you haven't already, consider checking out these pages:

* Explore strategies for minimizing mental complexity in the [Minimizing mental complexity](api-guidelines-minimizing-mental-complexity.md) page.
* Learn about maintaining backward compatibility in the [Backward compatibility](api-guidelines-backward-compatibility.md) page.
