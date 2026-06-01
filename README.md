# `rules_hugo`

[![Build Status](https://api.cirrus-ci.com/github/stackb/rules_hugo.svg)](https://cirrus-ci.com/github/stackb/rules_hugo)


<table><tr>
<td><img src="https://bazel.build/images/bazel-icon.svg" height="120"/></td>
<td><img src="https://raw.githubusercontent.com/gohugoio/hugoDocs/master/static/img/hugo-logo.png" height="120"/></td>
</tr><tr>
<td>Rules</td>
<td>Hugo</td>
</tr></table>

[Bazel](https://bazel.build) rules for building static websites with [Hugo](https://gohugo.io).

## Repository Rules

|               Name   |  Description |
| -------------------: | :----------- |
| [hugo_repository](#hugo_repository) | Load hugo dependency for this repo. |
| [github_hugo_theme](#github_hugo_theme) | Load a hugo theme from github. |

## Build Rules

|               Name   |  Description |
| -------------------: | :----------- |
| [hugo_site](#hugo_site) | Declare a hugo site. |
| [hugo_theme](#hugo_theme) | Declare a hugo theme. |

## Usage

### Add rules_hugo to your MODULE.bazel and add a theme

In your `MODULE.bazel`:

```python
bazel_dep(name = "rules_hugo_rmcguinness", version = "0.2.0")
# Required to load shell rules (like sh_test) under Bazel 9.1.0+
bazel_dep(name = "rules_shell", version = "0.6.1")

hugo_deps = use_extension("@rules_hugo_rmcguinness//hugo:extensions.bzl", "hugo_deps")

# Configure Hugo repository
hugo_deps.hugo_repository(
    name = "hugo",
    extended = True,
    version = "0.162.0",
)

# Load a theme from GitHub release archive
hugo_deps.http_archive(
    name = "hugo_theme_geekdoc",
    build_file_content = """
filegroup(
    name = "files",
    srcs = glob(["**"]),
    visibility = ["//visibility:public"]
)
    """,
    sha256 = "d53aca4bbcad45770b0b1e7bc03253b7b824270536578f2028966f68ba3a98d1",
    url = "https://github.com/thegeeklab/hugo-geekdoc/releases/download/v4.1.1/hugo-geekdoc.tar.gz",
)

use_repo(hugo_deps, "hugo", "hugo_theme_geekdoc")
```

### Declare a hugo_site with a theme in your BUILD file

```python
load("@rules_hugo_rmcguinness//hugo:rules.bzl", "hugo_serve", "hugo_site", "hugo_theme")
load("@rules_shell//shell:sh_test.bzl", "sh_test")

hugo_theme(
    name = "hugo_theme_geekdoc",
    theme_name = "hugo-geekdoc",
    srcs = [
        "@hugo_theme_geekdoc//:files",
    ],
)

# Note, here we are using the config_dir attribute to support multi-lingual configurations.
hugo_site(
    name = "site_complex",
    config_dir = glob(["config/**"]),
    content = glob(["content/**"]),
    data = glob(["data/**"]),
    quiet = False,
    theme = ":hugo_theme_geekdoc",
)

# Run local development server
hugo_serve(
    name = "serve",
    dep = [":site_complex"],
)

# Writing a test for your hugo_site
sh_test(
    name = "site_test",
    srcs = ["site_test.sh"],
    data = [":site_complex"],
)
```

### Previewing the site

Execute the following command:

```shell
bazel run //site_complex:serve
```

Then open your browser: [here](http://localhost:1313)


### Build the site

The `hugo_site` target emits the output in the `bazel-bin` directory.

```sh
$ bazel build //site_complex:site_complex
[...]
Target //site_complex:site_complex up-to-date:
  bazel-bin/site_complex/site_complex
[...]
```
```sh
$ tree bazel-bin/site_complex/site_complex
bazel-bin/site_complex/site_complex
├── 404.html
├── favicon.ico
[...]
```

### Testing the site

You can test that your Hugo site builds and outputs files successfully:

```sh
$ bazel test //site_simple:site_test
[...]
Target //site_simple:site_test up-to-date:
  bazel-bin/site_simple/site_test
INFO: Elapsed time: 0.198s
//site_simple:site_test                                                  PASSED
```

## End

See source code for details about additional rule attributes / parameters.
