---
title: "How I published links inside the pkg.go.dev documentation of my module"
date: 2024-10-26T01:18:13+01:00
categories: ['GoLang', 'documentation']
tags: ['godoc', 'emoji', 'go', 'golang', 'doc', 'documentation']
slug: links-godoc-module
twitter:
  card: summary
  site: ldez
  creator: ldez
  title: "How I published links inside the pkg.go.dev documentation of my module"
  image: /images/pkggodevlinks-package.png
  image_alt: "List of items inside the go doc with emojis"
---

While hunting down a package on https://pkg.go.dev, I stumbled upon a delightful surprise in the "Links" section: there are custom links!!

![](/images/pkggodevlinks-package.png)

What is this magic?

<!--more-->

Let's go to find if there is a documentation about that!

After some digging, I landed on the [About](https://pkg.go.dev/about) page and discovered a section called [Adding links](https://pkg.go.dev/about#adding-links).

But this section is a bit sparse about the topic: it's just a reference to the issue [42968](https://github.com/golang/go/issues/42968).

As an open source maintainer, I know that issues are not really the best documentation, the initial concept often morphs as ideas evolve.

I dove into the code itself and found even more than what the issue hinted at.

## ❓ How?

Note: The repository must have a license that follow the [license policy](https://pkg.go.dev/license-policy).

### readme.md syntax

```markdown
## Links

- [Title from readme 1](https://example.com)
- [Title from readme 2](https://example.org)
```

The heading level is not important, it can be `#`, `##`, `###`, etc.

### godoc syntax

```go
// Package foo Paneer fromage babybel.
// Ricotta paneer monterey jack parmesan cheesecake who moved my cheese bavarian bergkase cheeseburger. 
//
// Links
//
// - Title from godoc 1, https://example.com
// - Title from godoc 2, https://example.org
package foo
```

## 🪧 Where?

There are several ways to add links:

- the `readme.md` file
  - at the "root/module" level
  - inside any package level
- the package godoc
  - at the "root/module" level
  - inside any package level

## Hierarchy of the Links

```md
. (links from readme.md and godoc)
├── pkga (links from "root/module" readme.md)
├── pkgb (links from "root/module" readme.md + package readme.md)
├── pkgd (links from "root/module" readme.md + package godoc)
└── pkgc (links from "root/module" readme.md + package godoc + package readme.md)
```

At the "root/module" level, the links from `readme.md` and/or godoc will be displayed.

At the package level (not the "root/module" level):
- if you have links inside the `readme.md` of this package:
  - the links from the package `readme.md` (and from the "root/module" `readme.md`) will be displayed
- if you have link inside the godoc of this package:
  - the links from the godoc (and from the "root/module" `readme.md`) the package will be displayed
- if you have no package `readme.md` and no package godoc:
  - the links from the "root/module" `readme.md` will be displayed

The links from godoc are not transitives: the links from the parent package are not displayed.

The links from `readme.md` are partially transitives: only the "root/module" `readme.md` will be used.

## ✨ Results

### Root/Module

Links from `readme.md` and godoc:

[module page](https://pkg.go.dev/github.com/ldez/pkggodevlinks@v0.1.0)

![](/images/pkggodevlinks-package.png)

### Package bar

Links from "root/module" `readme.md` + package `readme.md`:

[bar package page](https://pkg.go.dev/github.com/ldez/pkggodevlinks@v0.1.0/bar)

![](/images/pkggodevlinks-bar-package.png)

### Package fii

Links from "root/module" `readme.md`:

[fii package page](https://pkg.go.dev/github.com/ldez/pkggodevlinks@v0.1.0/fii)

![](/images/pkggodevlinks-fii-package.png)

### Package foo

links from "root/module" `readme.md` + package godoc:

[foo package page](https://pkg.go.dev/github.com/ldez/pkggodevlinks@v0.1.0/foo)

![](/images/pkggodevlinks-foo-package.png)

### Package bir

Links from "root/module" `readme.md` + package godoc + package `readme.md`:

[bir package page](https://pkg.go.dev/github.com/ldez/pkggodevlinks@v0.1.0/bir)

![](/images/pkggodevlinks-bir-package.png)

## Why?

This can be useful if you want to add link to your documentation, your issue tracker, or more.

```markdown
## Links

- [❤️](https://github.com/sponsors/ldez)
- [📑 Documentation](https://go-acme.github.io/lego/)
- [🐞 Issues](https://github.com/go-acme/lego/issues)
```

## If you enjoyed this article

Please [give me a tip](https://github.com/sponsors/ldez) ❤️.
