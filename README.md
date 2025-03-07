![Maelstrom Logo (Dark Compatible)](https://github.com/maelstrom-software/maelstrom/assets/146376379/7b46a1c1-e67f-412a-b618-42f7e2c25139)

# `maelstrom-go-test` GitHub Action

This GitHub action installs and configures `maelstrom-go-test`, the Maelstrom
test runner for [Go](https://go.dev).
[Maelstrom](https://github.com/maelstrom-software/maelstrom) is a suite of
tools for running tests in isolated micro-containers locally on your machine or
distributed across arbitrarily large clusters. Maelstrom currently has test
runners for Rust, Go, and Python, with more on the way. For more information
about Maelstrom in general, please see the [main
repository](https://github.com/maelstrom-software/maelstrom), the [web
site](https://maelstrom-software.com), or the
[documentation](https://maelstrom-software.com/doc/book/latest).

# Usage

```yml
jobs:
  run-tests:
    name: Run Tests
    # Both ubuntu-22.04 and ubuntu-24.04 are supported.
    # Both x86-64 and ARM (AArch64) are supported.
    # The architecture needs to be the same as the workers.
    runs-on: ubuntu-24.04

    steps:
      # The broker needs to be installed and started before running the
      # maelstrom-go-test-action. The broker must be run in the same job as
      # maelstrom-go-test.
    - name: Install Maelstrom Broker
      uses: maelstrom-software/maelstrom-broker-action@v1

    - name: Start Maelstrom Broker
      run: |
        TEMPFILE=$(mktemp maelstrom-broker-stderr.XXXXXX)
        maelstrom-broker 2> >(tee "$TEMPFILE" >&2) &
        PID=$!
        PORT=$( \
          tail -f "$TEMPFILE" \
          | awk '/\<addr: / { print $0; exit}' \
          | sed -Ee 's/^.*\baddr: [^,]*:([0-9]+),.*$/\1/' \
        )
        echo "MAELSTROM_BROKER_PID=$PID" >> "$GITHUB_ENV"
        echo "MAELSTROM_BROKER_PORT=$PORT" >> "$GITHUB_ENV"
      env:
        MAELSTROM_BROKER_ARTIFACT_TRANSFER_STRATEGY: github

    - name: Schedule Post Handler to Kill Maelstrom Broker
      uses: gacts/run-and-post-run@v1
      with:
        post: kill -15 $MAELSTROM_BROKER_PID

      # This action installs and configures (via environment variables)
      # maelstrom-go-test so it can be run simply with `maelstrom-go-test`.
    - name: Install and Configure maelstrom-go-test
      uses: maelstrom-software/maelstrom-go-test-action@v1

    - name: Check Out Repository
      uses: actions/checkout@v4

      # You can now run `maelstrom-go-test` however you want.
    - name: Run Tests
      run: maelstrom-go-test

  # These are the worker jobs. Tests will execute on one of these workers.
  maelstrom-worker:
    strategy:
      matrix:
        worker-number: [1, 2, 3, 4]

    name: Maelstrom Worker ${{ matrix.worker-number }}

    # This must be the same architecture as the test-running job.
    runs-on: ubuntu-24.04

    steps:
    - name: Install and Run Maelstrom Worker
      uses: maelstrom-software/maelstrom-worker-action@v1
```

# How to Use

Use this action to install
[`maelstrom-go-test`](https://maelstrom-software.com/doc/book/latest/maelstrom-go-test.html)
and configure it (via environment variables). It can then be run however you
like in the rest of your job.

Before using this action, the
[`maelstrom-broker-action`](https://github.com/maelstrom-software/maelstrom-broker-action)
must be used, and the broker must be started, as shown in the example above.

There also must be at least one worker running in a separate job using the
[`maelstrom-worker-action`](https://github.com/maelstrom-software/maelstrom-worker-action),
as shows in the example above.

# What it Does

This action installs
[`maelstrom-go-test`](https://maelstrom-software.com/doc/book/latest/maelstrom-go-test.html)
and sets some environment variables so it will successfully connect with
broker. You are then free to run `maelstrom-go-test` however you like.

# Learn More

You can find documentation for running Maelstrom in GitHub
[here](https://maelstrom-software.com/doc/book/latest/github.html).

# Licensing

This project is available under the terms of either the Apache 2.0 license or the MIT license.
