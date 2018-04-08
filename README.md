tfnotify
========

[![][circleci-svg]][circleci] [![][codecov-svg]](codecov) [![][goreportcard-svg]][goreportcard]

[circleci]: https://circleci.com/gh/mercari/tfnotify/tree/master
[circleci-svg]: https://circleci.com/gh/mercari/tfnotify/tree/master.svg?style=svg
[codecov]: https://codecov.io/gh/mercari/tfnotify
[codecov-svg]: https://codecov.io/gh/mercari/tfnotify/branch/master/graph/badge.svg
[goreportcard]: https://goreportcard.com/report/github.com/mercari/tfnotify
[goreportcard-svg]: https://goreportcard.com/badge/github.com/mercari/tfnotify

tfnotify parses Terraform commands' execution result and applies it to an arbitrary template and then notifies it to GitHub comments etc.

## Motivation

There are commands such as `plan` and `apply` on Terraform command, but many developers think they would like to check if the execution of those commands succeeded.
Terraform commands are often executed via CI like Circle CI, but in that case you need to go to the CI page to check it.
This is very troublesome. It is very efficient if you can check it with GitHub comments or Slack etc.
You can do this by using this command.

<img src="./misc/images/1.png" width="600">

<img src="./misc/images/2.png" width="500">

## Installation

Grab the binary from GitHub Releases (Recommended)

or

```console
$ go get -u github.com/mercari/tfnotify
```


### What tfnotify does

1. Parse the execution result of Terraform
2. Bind parsed results to Go templates
3. Notify it to any platform (e.g. GitHub) as you like

Detailed specifications such as templates and notification destinations can be customized from the configration files (described later).

## Usage

### Basic

tfnotify is just CLI command. So you can run it from your local after grabbing the binary.

Basically tfnotify waits for the input from Stdin. So tfnotify needs to pipe the output of Terraform command like the following:

```console
$ terraform plan | tfnotify plan
```

For `plan` command, you also need to specify `plan` as the argument of tfnotify. In the case of `apply`, you need to do `apply`. Currently supported commands can be checked with `tfnotify --help`.

### Configurations

When running tfnotify, you can specify the configuration path via `--config` option (if it's omitted, it defaults to `{.,}tfnotify.y{,a}ml`).

The example settings of GitHub and Slack are as follows. Incidentally, there is no need to replace TOKEN string such as `$GITHUB_TOKEN` with the actual token. Instead, it must be defined as environment variables in CI settings.

[template](https://golang.org/pkg/text/template/) of Go can be used for `template`. The templates can be used in `tfnotify.yaml` are as follows:

Placeholder | Usage
---|---
`{{ .Title }}` | Like `## Plan result`
`{{ .Message }}` | A string that can be set from CLI with `--message` option
`{{ .Result }}` | Matched result by parsing like `Plan: 1 to add` or `No changes`
`{{ .Body }}` | The entire of Terraform execution result

#### Template Examples

<details>
<summary>For GitHub</summary>

```yaml
---
ci: circleci
notifier:
  github:
    token: $GITHUB_TOKEN
    repository:
      owner: "mercari"
      name: "tfnotify"
terraform:
  fmt:
    {{ .Title }}

    {{ .Message }}

    {{ .Result }}

    {{ .Body }}
  plan:
    template: |
      {{ .Title }}
      {{ .Message }}
      {{if .Result}}
      <pre><code> {{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>
      <pre><code> {{ .Body }}
      </pre></code></details>
  apply:
    template: |
      {{ .Title }}
      {{ .Message }}
      {{if .Result}}
      <pre><code> {{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>
      <pre><code> {{ .Body }}
      </pre></code></details>
```

</details>

<details>
<summary>For Slack</summary>

```yaml
---
ci: circleci
notifier:
  slack:
    token: $GITHUB_TOKEN
terraform:
  plan:
    template: |
      {{ .Message }}
      {{if .Result}}
      ```
      {{ .Result }}
      ```
      {{end}}
      ```
      {{ .Body }}
      ```
```

</details>

### Supported CI

Currently, supported CI are here:

- Circle CI
- Travis CI

## Committers

 * Masaki ISHIYAMA ([@b4b4r07](https://github.com/b4b4r07))

## Contribution

Please read the CLA below carefully before submitting your contribution.

https://www.mercari.com/cla/

## License

Copyright 2018 Mercari, Inc.

Licensed under the MIT License.
