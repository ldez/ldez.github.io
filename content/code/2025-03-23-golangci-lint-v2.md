---
title: "Welcome to golangci-lint v2"
date: 2025-03-23T01:18:13+01:00
categories: ['GoLang']
tags: ['golangci-lint', 'golangci', 'linting', 'linter', 'go', 'golang']
slug: golangci-lint-v2
twitter:
  card: summary
  site: ldez
  creator: ldez
  title: "Welcome to golangci-lint v2"
  image: images/amazing-world-golangci-lint-v2.png
  image_alt: "The amazing world of golangci-lint v2"
---

I am thrilled to announce the release of v2!

I hope is that these features will enhance your development experience.

![](/images/amazing-world-golangci-lint-v2.png)

<!--more-->

## 📄 Configuration Overhaul

One of the most significant updates in golangci-lint v2 is the revamped configuration structure.

### 🧩 Simplified Linters Management

The `enable-all` and `disable-all` options have been replaced with a single, more intuitive option: `linters.default`.

```yaml
linters:
  default: all # standard/all/none/fast
```

To address the confusion surrounding the previous `fast` option, v2 introduces two distinct options for using "fast" linters:

1. Fast linters as default: you can now set fast linters as the default set of linters using the `linters.default: fast` option.
   This ensures that only the fastest linters are used by default.
2. Fast linters filter: the `--fast-only` flag allows you to filter all the linters defined in your configuration to keep only the fast linters.
   This option provides a quick way to focus on performance-critical linters without modifying your configuration file.

The linter settings have moved to the `linters` section:

```yaml
linters:
  default: standard
  enable:
    - misspell
  settings:
    misspell:
      locale: US
      extra-words:
        - typo: "iff"
          correction: "if"
```

### 📁 Improved File Paths Management

All options related to file paths are now relative to the configuration file path by default, rather than the location where the binary is launched.
This change ensures consistency and reduces confusion.

This behavior is configurable:

```yaml
run:
  relative-path-mode: cfg # cfg, wd, gomod, gitroot.
```

Inside some linters configuration that use paths,
you can now use the placeholder `${base-path}` to refer to the same default location as the golangci-lint options.

{{< details >}}

```yaml
linters:
  settings:
    gocritic:
      settings:
        ruleguard:
          rules: ${base-path}/ruleguard/rules-*.go,${base-path}/myrule1.go
```

{{< /details >}}

### 🙈 Issue Exclusions

In v2, there are no exclusions by default.

However, you now have the possibility to enable human-readable exclusion presets.

```yaml
linters:
  exclusions:
    presets:
      - comments
      - std-error-handling
      - common-false-positives
      - legacy
```

The unused exclusion rules can be detected and reported as warnings.

```yaml
linters:
  exclusions:
    warn-unused: true
    rules:
      - path: path/to/file.go
        linters:
          - gocyclo
```


## 🌟 Introducing the `golangci-lint fmt` Command

golangci-lint v2 introduces a new command, `golangci-lint fmt`, designed to format your code using your preferred formatter or a combination of formatters.

This command adds a new layer of flexibility to your code formatting process.

A new section, `formatters`, has been added to the configuration file.

This section allows you to specify and configure the formatters used by the `golangci-lint fmt` command, giving you full control over how your code is formatted.

The formatters defined in this section are automatically used as "linter" when you run the command `golangci-lint run`.

```yaml
formatters:
  enable:
    - gofmt
    - goimports
  settings:
    gofmt:
      simplify: false
      rewrite-rules:
        - pattern: interface{}
          replacement: any
        - pattern: a[b:len(a)]
          replacement: a[b:]
```


## ♻️ Seamless Migration

To ease the transition to v2, golangci-lint includes a migration command: `golangci-lint migrate`.
This command automatically converts your v1 configuration to the new v2 format, ensuring a smooth and hassle-free upgrade process.

We also created a detailed [migration guide](https://golangci-lint.run/product/migration-guide/).

## ❤️ Enjoy

For more key updates, see the [changelog](https://opencollective.com/redirect?url=https%3A%2F%2Fgolangci-lint.run%2Fproduct%2Fchangelog%2F%23v200).

This major release marks an exciting new chapter in the evolution of golangci-lint.

Golangci-lint, like many other open-source projects, relies on the generosity of its community through donations and corporate sponsorships to support its growth and development.

So consider donating to the project, as giving back through donations is a wonderful way to say thank you for the work we do.

- https://donate.golangci.org
- https://donate.ldez.dev

Thank you for being a part of this community, and happy linting with golangci-lint v2!
