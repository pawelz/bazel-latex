# Bazel rules for LaTeX

This repository provides [Bazel](https://bazel.build/) rules for LaTeX,
inspired by [Klaus Aehlig's blog post](http://www.linta.de/~aehlig/techblog/2017-02-19.html)
on the matter.

Instead of depending on the host system's copy of LaTeX, these rules
download [a modular copy of TeXLive from GitHub](https://github.com/ProdriveTechnologies/texlive-modular).
By using fine-grained dependencies, you will only download portions of
TeXLive that are actually used in your documents.

As the output of the LaTeX tools is unnecessarily verbose, these build
rules invoke LaTeX using [latexrun](https://github.com/aclements/latexrun).
Errors and warnings are formatted similar to those generated by Clang.

# Using these rules

Add the following to `WORKSPACE`:

```python
http_archive(
    name = "bazel_latex",
    sha256 = "<checksum>",
    strip_prefix = "bazel-latex-<release>",
    url = "https://github.com/ProdriveTechnologies/bazel-latex/archive/v<release>.tar.gz",
    patches = ["some_patch.patch"], % Optional
)

load("@bazel_latex//:repositories.bzl", "latex_repositories")

latex_repositories()
```

And add the following `load()` directive to your `BUILD` files:

```python
load("@bazel_latex//:latex.bzl", "latex_document")
```

You can then use `latex_document()` in `BUILD` file to declare documents that need to be
built.

```python
latex_document(
    name = "my_report",
    srcs = glob([
        "chapters/*.tex",
        "figures/*",
        "references.bib",
    ]) + ["//company_dir:company_style"],
    cmd_flags = ["--any-additional-flags"], % Optional
    main = "my_report.tex",
)
```

Utilize `cmd_flags` to provide optional command line arguments.

Commonly reused sources (e.g., templates) can be placed in
[`filegroup()`](https://docs.bazel.build/versions/master/be/general.html#filegroup)
blocks, so that they don't need to be repeated. Those `filegroup()` could
be located not just in the single `BUILD` file, but in any of sub directories.
For example, if you want to include company specific template files which are
located in `//company_dir` directory as `company_style`, then declare them as
like following in `company_dir/BUILD` file, and include the dependency, like
`//company_dir:company_style`, in `latex_repositories`.

```python
filegroup(
    name = "company_style",
    srcs = glob([
        ...
    ]),
)
```

A PDF can be built by running:

```
bazel build //example:my_report
```

It can be viewed using your system's PDF viewer by running:

```
bazel run //example:my_report_view
```

If you want to get the output from the PDF viewer you can run:

```
bazel run //example:my_report_view_output
```

# Using packages

By default, `latex_document()` only provides a version of TeXLive that
is complete enough to build the most basic documents. Whenever you use
`\usepackage{}` in your documents, you must also add a corresponding
dependency to your `latex_document()`. This will cause Bazel to download
and expose those packages for you. Below is an example of how a document
can be built that depends on the Hyperref package.

```python
latex_document(
    name = "hello",
    srcs = ["@bazel_latex//packages:hyperref"],
    main = "hello.tex",
)
```

This repository provides bindings for most commonly used packages.
Please open a pull request if additional bindings are needed.

## Local packages
If the desired package to use is not available through bazel-latex, but is
available in TeX Live, then it is possible to patch `BUILD.bazel` in
`/packages` to add support for the desired package locally.

Therefore, clone bazel-latex locally, and make the desired changes to the 
packages build file. Then, put the output of the diff in `some_patch.patch`,
and update your `WORKSPACE` accordingly as shown below.

```python
http_archive(
    name = "bazel_latex",
    sha256 = "<checksum>",
    strip_prefix = "bazel-latex-<release>",
    url = "https://github.com/ProdriveTechnologies/bazel-latex/archive/v<release>.tar.gz",
    patches = ["some_patch.patch"],
)
```

If this solution does not suffice, please feel free to open a PR to add the corresponding package to Bazel LaTeX. In that case, also see `CONTRIBUTING.md`.

# Example

An example is available in the corresponding folder. The example can
be executed by running:
```
bazel run //example:my_report_view
```

# Platform support

These rules have been tested to work on (using bazel 3.2.0):

- FreeBSD 11.2, building locally.
- macOS Mojave 10.14, building locally.
- macOS Catalina 10.15, building locally.
- Ubuntu 18.04, building locally.
- Ubuntu 18.04 WSL, building locally.
- Ubuntu 18.04, building on a Debian 8 based
  [Buildbarn](https://github.com/buildbarn) setup.
- Ubuntu 19.04 (Disco Dingo), building locally.
- Ubuntu 20.04, building locally.
- Manjaro 18.1.2 (Juhraya), building locally.
- Windows 10 1803, building on a Debian 8 based
  [Buildbarn](https://github.com/buildbarn) setup.
- Raspbian 11.3, building locally.

These rules are known not to work on:

- Windows 10 1803, building locally.
