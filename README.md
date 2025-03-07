![Maelstrom Logo (Dark Compatible)](https://github.com/maelstrom-software/maelstrom/assets/146376379/7b46a1c1-e67f-412a-b618-42f7e2c25139)

# Maelstrom Broker GitHub Action

This GitHub action installs and runs the Maelstrom broker.
[Maelstrom](https://github.com/maelstrom-software/maelstrom) is a suite of
tools for running tests in isolated micro-containers locally on your machine or
distributed across arbitrarily large clusters. Maelstrom currently has test
runners for Rust, Go, and Python, with more on the way. For more information
about Maelstrom in general, please see the [main
repository](https://github.com/maelstrom-software/maelstrom), the
[web site](https://maelstrom-software.com), or the
[documentation](https://maelstrom-software.com/doc/book/latest).

# Usage

```yml
jobs:
  run-tests:
    name: Run Tests
    runs-on: ubuntu-24.04

    steps:
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

    - name: Check Out Repository
      uses: actions/checkout@v4

    - name: Install maelstrom-go-test
      uses: maelstrom-software/maelstrom-go-test-action@v1

    - name: Run Tests
      run: cargo maelstrom
```

# How to Use

Run at the beginning of a job that will use Maelstrom test runner, 
like `cargo-maelstrom`, `maelstrom-go-test`, or `maelstrom-pytest`. Maelstrom
workers should be run on separate jobs. See [this
example workflow](https://github.com/maelstrom-software/maelstrom-examples/blob/main/.github/workflows/ci-base.yml).

# What it Does

This action installs
[`maelstrom-broker`](https://maelstrom-software.com/doc/book/latest/broker.html),
and exposes some environment variables needed for later actions.

Unfortunately, because of limitations with GitHub actions, this action doesn't
start the broker. You need to do that yourself, as shown in the example.
Additionally, you should set a post handler to kill the broker at the end of
the job, also show in the example.

# Learn More

You can find documentation for running Maelstrom in GitHub
[here](https://maelstrom-software.com/doc/book/latest/github.html).

# Licensing

This project is available under the terms of either the Apache 2.0 license or the MIT license.
