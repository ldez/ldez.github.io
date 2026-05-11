---
title: "Welcome to go-acme/lego v5"
date: 2026-05-11T01:06:13+01:00
categories: ['GoLang']
tags: ['acme', 'certificate', 'tls', 'go', 'golang']
slug: lego-v5
twitter:
  card: summary
  site: ldez
  creator: ldez
  title: "Welcome to go-acme/lego v5"
  image: images/lego-logo.min.svg
  image_alt: "go-acme/lego v5"
---

I am thrilled to announce go-acme/lego v5!

This is a major release that brings a completely redesigned CLI, and powerful new features.

![](/images/lego-logo.min.svg)

<!--more-->

This release asks me a lot of work (~150 pull requests during 3 months, 100% Human work).

## 🚀 What's New

### 💍 One Command to Rule Them All: `lego run`

The biggest CLI change in v5 is the unification of `run` and `renew` into a single command: `lego run`.

It obtains a new certificate if none exists and renews it when it's time.
No more juggling two commands.

Flags have also moved: global flags are now command-level flags for clarity.

```bash
# Before (v4)
lego --dns cloudflare -d '*.example.com' -d example.com run

# After (v5)
lego run --dns cloudflare -d '*.example.com' -d example.com
```

See the [documentation](https://go-acme.github.io/lego/obtain/) for more details.

### 📄 Configuration File Support

You can now drive lego entirely from a `.lego.yml` file, eliminating long command lines and making automation easier.

```yaml
challenges:
  cf:
    dns:
      provider: cloudflare

certificates:
  my-cert:
    challenge: cf
    domains:
      - example.com
      - '*.example.com'
```

Then just run:

```bash
CLOUDFLARE_EMAIL="you@example.com" \
CLOUDFLARE_API_KEY="yourkey" \
lego
```

You can also use [dotenv files](https://go-acme.github.io/lego/dns/#dotenv-file) to manage your credentials.

The configuration file supports everything: certificates, challenges, accounts, servers, hooks, and logging.

It can be validated by a [JSON Schema](https://go-acme.github.io/lego/lego.jsonschema.json).

See the [documentation](https://go-acme.github.io/lego/references/ref-file/) for more details.

### 🐙 New Commands for Account, Certificate, and Archive Management

v5 introduces dedicated subcommands for managing your lego data:

Accounts:

- `lego accounts register`: Register a new ACME account.
- `lego accounts recover`: Recover/import an existing account from a private key.
- `lego accounts keyrollover`: Rotate the account private key.
- `lego accounts list`: List all accounts managed by lego.

See the [documentation](https://go-acme.github.io/lego/advanced/accounts/) for more details.

Certificates:

- `lego certificates list`: List all certificates with their status and expiration date.
- `lego certificates revoke`: Revoke one or all certificates.

See the [documentation](https://go-acme.github.io/lego/advanced/certificates/) for more details.

Archives:

- `lego archives list`: List all backed-up accounts and certificates.
- `lego archives restore`: Restore a backup.

See the [documentation](https://go-acme.github.io/lego/advanced/archives/) for more details.

### 🔒 DNS-PERSIST-01 Challenge

lego now supports the new `dns-persist-01` challenge type.

WARNING:
- The RFC is still a draft.
- This is currently not available in most CA production.

```bash
lego run -d 'example.com' --dns-persist
```

See the [documentation](https://go-acme.github.io/lego/obtain/dnspersist01/) for more details.

### 🧠 Smarter Certificate Renewal

EAB (External Account Binding) credentials are no longer required at renewal time, only at initial registration.

This simplifies automated renewal pipelines, especially with commercial CAs.

### 🪝 Pre-Hook, Deploy-Hook, and Post-Hook

lego v5 introduces three lifecycle hooks to let you run scripts around certificate issuance:

| Hook          | When it runs                                                                         |
|---------------|--------------------------------------------------------------------------------------|
| `pre-hook`    | Before the certificate is created or renewed (only if a change will actually happen) |
| `deploy-hook` | After the certificate is successfully created or renewed                             |
| `post-hook`   | After the operation completes, regardless of outcome                                 |

```bash
lego run -d 'example.com' --deploy-hook='./my-deploy-hook.sh'
```

Hooks receive rich context via environment variables (`LEGO_HOOK_CERT_PATH`, `LEGO_HOOK_CERT_KEY_PATH`, etc.).

With a Configuration File:

```yaml
hooks:
  pre:
    command: './my-pre-hook.sh'
  deploy:
    command: './my-deploy-hook.sh'
  post:
    command: './my-post-hook.sh'
```

See the [documentation](https://go-acme.github.io/lego/advanced/hooks/) for more details.

Don't hesitate to share your hook scripts with [the community](https://github.com/go-acme/lego/discussions/categories/ideas).

### 🌐 IPv6-Only Support

For hosts running on IPv6-only networks, lego v5 can be configured to exclusively use the IPv6 network stack.

```bash
lego run -d 'example.com' --http --ipv6only
```

With a Configuration File:

```yaml
networkStack: ipv6only
```

### 📰 Structured Logging with JSON Output

lego v5 introduces structured logging with support for `text`, `colored` (default), and `json` formats (useful for log collectors).


```bash
lego --log.format=json --log.level=info run -d 'example.com' --http
```

Note that `--log.format` and `--log.level` are global flags.

With a Configuration File:

```yaml
log:
  level: info
  format: json
```

### 🏷️ CA Server Short-Codes

Instead of typing full ACME server URLs, you can now use short-codes for well-known CAs:

```bash
lego run --server='letsencrypt-staging' ...
lego run --server='zerossl' ...
lego run --server='googletrust' ...
```

A full list of supported short-codes is available in the [documentation](https://go-acme.github.io/lego/advanced/caservers/).

### 🗃️ 24 New DNS Providers

lego v5 adds support for 24 new DNS providers, bringing the total to over 210:

[51DNS](https://go-acme.github.io/lego/dns/dns51/), [Abion](https://go-acme.github.io/lego/dns/abion/), [Curanet](https://go-acme.github.io/lego/dns/curanet/), [DanDomain](https://go-acme.github.io/lego/dns/dandomain/), [Dinahosting](https://go-acme.github.io/lego/dns/dinahosting/), [DNS.services](https://go-acme.github.io/lego/dns/dnsservices/), [DNScale](https://go-acme.github.io/lego/dns/dnscale/), [dnsla](https://go-acme.github.io/lego/dns/dnsla/), [EUsrv](https://go-acme.github.io/lego/dns/euserv/), [Fornex](https://go-acme.github.io/lego/dns/fornex/), [Gehirn](https://go-acme.github.io/lego/dns/gehirn/), [Gname](https://go-acme.github.io/lego/dns/gname/), [Katapult](https://go-acme.github.io/lego/dns/katapult/), [NederHost](https://go-acme.github.io/lego/dns/nederhost/), [NGENIX](https://go-acme.github.io/lego/dns/ngenix/), [omg.lol](https://go-acme.github.io/lego/dns/omglol/), [PointDNS/PointHQ](https://go-acme.github.io/lego/dns/pointdns/), [Rage4](https://go-acme.github.io/lego/dns/rage4/), [ScanNet](https://go-acme.github.io/lego/dns/scannet/), [Tele3](https://go-acme.github.io/lego/dns/tele3/), [Veesp](https://go-acme.github.io/lego/dns/veesp/), [Wannafind](https://go-acme.github.io/lego/dns/wannafind/), [Xinnet](https://go-acme.github.io/lego/dns/xinnet/), [Zilore](https://go-acme.github.io/lego/dns/zilore/).

See the [documentation](https://go-acme.github.io/lego/dns/) for more details.

## ♻️ Migrating from v4

v5 includes breaking changes to the CLI, directory structure, and the API of the library.

Please run the new v5 command `lego migrate` before running any other commands.

```bash
lego migrate
```

or

```bash
lego migrate --path /path/to/lego/storage
```

This migrates your storage directory to the new layout.

See the full [migration guide](https://go-acme.github.io/lego/migration/) for details on flags, environment variables, and other changes.

## 📦 Get lego v5

Download the latest release from the [GitHub releases page](https://github.com/go-acme/lego/releases) or use your [preferred package manager](https://go-acme.github.io/lego/install/).

We'd love to hear your feedback.

## ❤️ Support lego

lego is an independent, free, and open-source project.

It takes a lot of time and effort to maintain: Maintaining lego is maintaining an ACME client library, a CLI, and about +200 DNS implementations.

If you find lego useful, please consider [supporting me](https://donate.ldez.dev).

If you are a company, we have [dedicated tiers](https://github.com/sponsors/go-acme).

Every contribution, however small, makes a real difference.

Thank you!
