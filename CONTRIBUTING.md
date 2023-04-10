# Contribute documentation

If you wish to contribute to the documentation, please read through [Kotlin documentation guidelines][1]

## Documentation format

Our documentation is written in Markdown format with some domain specific language (DSL) constructs that are used at
JetBrains.

Unfortunately, this means that the document might not be rendered correctly or as expected when viewing it on GitHub or 
locally, as some tags and constructs work differently or are very specific.

For example, you might see this and think it to be a bug, because, normally, the `<img>` tag expects the `src` attribute 
to have a relative link to the image file:

```markdown
<img src="debug-person-builder.png"/>
```

However, in our case, it's not a bug, but a feature - it's enough to specify the file name only, and our documentation
engine will link it correctly with the actual image file correctly.

You can find this and other formatting rules in the [Kotlin documentation guidelines][1] document.

[1]: https://docs.google.com/document/d/1mUuxK4xwzs3jtDGoJ5_zwYLaSEl13g_SuhODdFuh2Dc/edit?usp=sharing
