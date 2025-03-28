# junit2json-rs

[![CI](https://github.com/Kesin11/junit2json-rs/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Kesin11/junit2json-rs/actions/workflows/ci.yml)
[![Crates.io](https://img.shields.io/crates/v/junit2json.svg)](https://crates.io/crates/junit2json)

junit2json-rs is a tool to convert JUnit XML format to JSON.
From a library perspective, it provides a function to serialize Junit XML to Struct.

junit2json-rs is a reimplementation of [ts-junit2json](https://github.com/Kesin11/ts-junit2json) that is my previous work in TypeScript.

# Purpose

junit2json-rs is designed for uploading test result data to BigQuery or any other DB that supports JSON.

Many languages and test frameworks support to output test result data as JUnit XML format, which is de fact standard in today.
On the other hand, most DBs do not support to import XML but support JSON.

For this purpose, junit2json-rs provides a simple JUnit XML to JSON converter.

# Install

```shell
cargo install junit2json
```

# Usage

```shell
junit2json -p <junit_xml_file>
```

# Output example

```json
{
  "testsuites": {
    "name": "gcf_junit_xml_to_bq_dummy",
    "time": 8.018,
    "tests": 12,
    "failures": 2,
    "testsuite": [
      {
        "name": "__tests__/gen_dummy_junit/dummy1.test.js",
        "tests": 4,
        "failures": 1,
        "errors": 0,
        "time": 4.772,
        "skipped": 0,
        "timestamp": "2020-01-12T16:33:13",
        "testcase": [
          {
            "name": "dummy1 Always success tests should be wait 0-2sec",
            "classname": "dummy1 Always success tests should be wait 0-2sec",
            "time": 0.414
          },
          {
            "name": "dummy1 Always success tests should be wait 1-3sec",
            "classname": "dummy1 Always success tests should be wait 1-3sec",
            "time": 1.344
          },
          {
            "name": "dummy1 Randomly fail tests should be wait 0-1sec and fail 50%",
            "classname": "dummy1 Randomly fail tests should be wait 0-1sec and fail 50%",
            "time": 0.673,
            "failure": {
              "inner": "Error: expect(received).toBeGreaterThan(expected)\n\nExpected: > 50\nReceived:   4.897277513425746\n    at Object.it (/Users/kesin/github/gcf_junit_xml_to_bq/__tests__/gen_dummy_junit/dummy1.test.js:22:17)"
            }
          },
          {
            "name": "dummy1 Randomly fail tests should be wait 1-2sec and fail 30%",
            "classname": "dummy1 Randomly fail tests should be wait 1-2sec and fail 30%",
            "time": 1.604
          }
        ]
      },
      {
        "name": "__tests__/gen_dummy_junit/dummy3.test.js",
        "tests": 4,
        "failures": 1,
        "errors": 0,
        "time": 6.372,
        "skipped": 0,
        "timestamp": "2020-01-12T16:33:13",
        "testcase": [
          {
            "name": "dummy3 Always success tests should be wait 0-2sec",
            "classname": "dummy3 Always success tests should be wait 0-2sec",
            "time": 1.328
          },
          {
            "name": "dummy3 Always success tests should be wait 1-3sec",
            "classname": "dummy3 Always success tests should be wait 1-3sec",
            "time": 2.598
          },
          {
            "name": "dummy3 Randomly fail tests should be wait 0-1sec and fail 30%",
            "classname": "dummy3 Randomly fail tests should be wait 0-1sec and fail 30%",
            "time": 0.455,
            "failure": {
              "inner": "Error: expect(received).toBeGreaterThan(expected)\n\nExpected: > 30\nReceived:   12.15901879426653\n    at Object.it (/Users/kesin/github/gcf_junit_xml_to_bq/__tests__/gen_dummy_junit/dummy3.test.js:22:17)"
            }
          },
          {
            "name": "dummy3 Randomly fail tests should be wait 1-2sec and fail 20%",
            "classname": "dummy3 Randomly fail tests should be wait 1-2sec and fail 20%",
            "time": 1.228
          }
        ]
      }
    ]
  }
}
```

# With `jq` examples

Show testsuites test count

```
junit2json <junit_xml_file> | jq .testsuites.tests
```

Show testsuite names

```
junit2json <junit_xml_file> | jq .testsuites.testsuite[].name
```

Show testcase classnames

```
junit2json <junit_xml_file> | jq .testsuites.testsuite[].testcase[].classname
```

# Notice

> [!IMPORTANT]
> junit2json-rs has some major changes from ts-junit2json.
> Most of the changes are to compliant with the JUnit XML Schema.

- A `testsuites` or `testsuite` key appears in the root of JSON.
- `properties` has `property` array. ts-junit2json has `property` array of object directly.
- `skipped`, `error`, `failure` are object, not array of object.
- If XML has undefined tag, it will be ignored. ts-junit2json will be converted to JSON if possible.

Referenced JUnit XML Schema:

- <https://llg.cubic.org/docs/junit/>
- <https://github.com/testmoapp/junitxml/tree/main>

# CLI Options

```
A tool convert JUnit XML format to JSON with Rust

Usage: junit2json [OPTIONS] <PATH>

Arguments:
  <PATH>  JUnit XML path

Options:
  -p, --pretty                     Output pretty JSON
  -f, --filter-tags <FILTER_TAGS>  Filter XML tag names [possible values: system-out, system-err]
  -h, --help                       Print help
  -V, --version                    Print version
```

# WASI

junit2json-rs also provides WASI executable.

If you have wasm runtime (ex. wasmtime), you can execute `junit2json.wasm` that can download from [GitHub Releases](https://github.com/Kesin11/junit2json-rs/releases) instead of native binary.

```shell
wasmtime --dir=. junit2json.wasm -- -p <junit_xml_file>
```

# Verify attestation

We addd attestations to released binary. So everyone can verify binary attestations using `gh attestation verify`.

ref: https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds#verifying-artifact-attestations-with-the-github-cli

Example commands for verify artifact that released v0.2.10

```bash
gh release download v0.2.10
tar -xzvf junit2json-rs-0.2.10-aarch64-apple-darwin.tar.gz

gh attestation verify junit2json -R Kesin11/junit2json-rs
```

# Development

## Setup

You can use DevContainer or Codespaces. Please see [devcontainer.json](./.devcontainer/devcontainer.json).

## Build

Build native binary

```bash
cargo build
cargo build --release
```

Build WASI

```bash
rustup target add wasm32-wasip1
cargo build --target wasm32-wasip1 --release
```

## Test

```bash
# Run test
cargo nextest run

# Update snapshot
## Need to install cargo-insta first.
## `cargo install cargo-insta`
cargo insta review
```

# License

MIT
