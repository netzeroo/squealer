![Sqealer](squealer.png)

# Squealer

### Telling tales on you for leaking secrets!

[![Build Status](https://travis-ci.com/owenrumney/squealer.svg?branch=main)](https://travis-ci.com/owenrumney/squealer)
[![codecov](https://codecov.io/gh/owenrumney/squealer/branch/main/graph/badge.svg?token=2EH55OCCX7)](https://codecov.io/gh/owenrumney/squealer)
[![Go Report Card](https://goreportcard.com/badge/github.com/owenrumney/squealer)](https://goreportcard.com/report/github.com/owenrumney/squealer)
[![Github Release](https://img.shields.io/github/release/owenrumney/squealer.svg)](https://github.com/owenrumney/squealer/releases)
[![GitHub All Releases](https://img.shields.io/github/downloads/owenrumney/squealer/total)](https://github.com/owenrumney/squealer/releases)

Squealer scans a local git repository for secrets that are being leaked deep within the commit history. 

The built-in configuration has the following checks;

AWS
- access key id
- access secret key

Github
- github token

Slack
- slack token OAUTH
- webhook url


Other
- Asymmetric Private Key

Sometimes we have secrets committed to our projects, generally we can invalidate them and move on. If squealer is telling tales about a secret that you are aware of and has been mitigated, you can use the `exception` rule found in the output to register it as ignored.

## Installation

curl -s "https://raw.githubusercontent.com/owenrumney/squealer/main/scripts/install.sh" | bash


## Usage

Squealer is intended to be run either locally or as part of a CI process. 

```shell
./squealer --help
Telling tales on your secret leaking

Usage:
  squealer [flags]

Flags:
      --concise                Reduced output.
      --config-file string     Path to the config file with the rules.
      --debug                  Include debug output.
      --everything             Scan all commits.... everywhere.
      --from-hash string       The hash to work back to from the starting hash.
  -h, --help                   help for squealer
      --no-git                 Scan as a directory rather than a git history.
      --output-format string   The format that the output should come in (default, json, sarif.
      --redacted               Display the results redacted.
      --to-hash string         The most recent hash to start with.
```

### Config File

```yaml
rules:
- rule: (A3T[A-Z0-9]|AKIA|AGPA|AIDA|AROA|AIPA|ANPA|ANVA|ASIA)[A-Z0-9]{16}
  description: Check for AWS Access Key Id
- rule: (?i)aws(.{0,20})?(?-i)['\"][0-9a-zA-Z\/+]{40}['\"]
  description: Check for AWS Secret Access Key
- rule: (?i)github[_\-\.]?token[\s:,="\]']+?(?-i)[0-9a-zA-Z]{35,40}
  description: Check for Github Token 
- rule: https://hooks.slack.com/services/T[a-zA-Z0-9_]{8}/B[a-zA-Z0-9_]{8}/[a-zA-Z0-9_]{24}
  description: Check for Slack webhook
- rule: xox[baprs]-([0-9a-zA-Z]{10,48})?
  description: Check for Slack token
- rule: '-----BEGIN ((EC|PGP|DSA|RSA|OPENSSH) )?PRIVATE KEY( BLOCK)?-----'
  description: Check for Private Asymetric Key
ignore_paths:
- vendor
- node_modules
ignore_extensions:
- .zip
- .png
- .jpg
- .pdf
- .xls
- .doc
- .docx
exceptions:
- exception: release/update.go:D2IDetI6aidl58GE6dv5uAaWmXM=
  reason: This is a webhook that we got rid of - can be ignored in this file
```

### Config breakdown

The config file is made up of the `rules`, `ignore_prefixes`, `ignore_extensions` and `exceptions`. 

#### rules

Rules define the regular expression that is used to detect the secret. Requires a description for posterity.

#### ignore_paths

Ignore paths are folders that you don't want to look ing - generally `vendor` and the like.

#### ignore_extensions

Ignore extensions have the file types that won't be scanned. Binaries are automatically ignored.

#### exceptions

Exceptions are the entries that you've already handled and don't want to be reported any more.

## Example Output

```shell
INFO[0000] Using a git scanner to process ../../tfsec/tfsec
INFO[0000] starting at hash 3bd04e7e17f2aad9e5f38826d88325798534a289

Content:      | access_key = "AKIAABCD12ABCDEF1ABC"
Filename:     | internal/app/tfsec/checks/aws044.go
Line No:      | 21
Secret Hash:  | bcE9jU2WV11OYs63eGHPZf1l9v8=
Commit:       | 4e68e1c5b3bc66982e4b7e6c5cc1c1642c87f83d
Committer:    | GitHub (noreply@github.com)
Committed:    | 2020-10-21 21:59:22 +0100 +0100
Exclude rule: | internal/app/tfsec/checks/aws044.go:bcE9jU2WV11OYs63eGHPZf1l9v8=

Content:      | access_key = "AKIAABCD12ABCDEF1ABC"
Filename:     | docs-website/docs/aws/AWS044.md
Line No:      | 26
Secret Hash:  | bcE9jU2WV11OYs63eGHPZf1l9v8=
Commit:       | 8a7715f2cf5a2ac74a1e186792c476fd52ee1474
Committer:    | ¨Owen Rumney (owen.rumney@form3.tech)
Committed:    | 2021-01-24 19:04:27 +0000 +0000
Exclude rule: | docs-website/docs/aws/AWS044.md:bcE9jU2WV11OYs63eGHPZf1l9v8=

Processing:
  duration:     2.99s
  commits:      503
  commit files: 4095

transgressionMap:
  identified:   6
  ignored:      0
  reported:     2


INFO[0002] Exit code: 1

```

It's worth noting that these are known because they're examples in the documentation for tfsec - I can add them to the `config.yaml` as exclusions y using the `Exclude rule`

## Credits

[Image by Derangedmisfit](https://derangedmisfit.newgrounds.com/)
