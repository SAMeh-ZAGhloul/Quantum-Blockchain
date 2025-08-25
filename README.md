# Quantum Secured Blockchain

**Note:** this is forked from: https://github.com/Quantum-Blockchains/quantum-metachain (with minor updates for rust packages ... see "**Build & Vendoring fixes (developer notes)**" section below)

This is a repository for Quantum Meta-chain, an implementation of a quantum node using quantum and post-quantum security. It is a fork of a rust-based repository, [Substrate](https://github.com/paritytech/substrate).

## Table of contents

- [1. Setup](#1-setup)
  - [1.1. Prerequisites](#11-prerequisites)
- [2. Building](#2-build)
  - [2.1. Using `cargo`](#21-using-cargo)
  - [2.2. Using Docker](#22-using-docker)
- [3. Running](#3-running)
  - [3.1. Using Python](#31-using-python)
  - [3.2. Using Docker](#32-using-docker)
- [4 - Testing](#4-testing)
  - [4.1 Runner unit tests](#41-runner-unit-tests)
  - [4.2 Rust unit tests](#42-rust-unit-tests)
  - [4.3 Key rotation tests](#43-key-rotation-tests)
- [5 - Documentation](#5-documentation)
- [6. The whitepaper](#6-the-whitepaper)
- [Build &amp; Vendoring fixes (developer notes)](#build--vendoring-fixes-developer-notes)

## 1. Setup

### 1.1. Prerequisites

To begin working with this repository you will need the following dependencies:

- [Rust](https://www.rust-lang.org/tools/install)
- [Python](https://www.python.org/downloads/)
- [Docker](https://docs.docker.com/engine/install/) (optional)
- QKD-simulator
- Certificate for QKD-simulator

After downloading your dependencies you need to make sure to continue with these steps:

- Because this a substrate fork you will also need to configure Rust with few additional steps, listed [here](https://docs.substrate.io/install/)
  by substrate team.
- Install Python dependencies:

```bash
python3 -m venv venv
. ./venv/bin/activate
pip3 install -r requirements.txt
```

## 2. Build

There are few ways to build this repository before running, listed below.

### 2.1. Using `cargo`

Cargo is a tool provided by rust framework to easily manage building, running and testing rust code.
You can use it to build quantum node code with command:

```bash
cargo build --release
```

This will create a binary file in `./target/release`, called `qmc-node`.

### 2.2. Using Docker

Alternate way of building this repository uses Docker. To build a node use command:

```bash
make build
```

This will create a `quantum-metachain` docker image.

## 3. Running

Depending on how you built your project you can run it in different ways

Before you start the node, you need to create a configuration file, the path to which must then be provided when you start the node.
Example:

```json
{
  "__type__": "Config",
  "local_peer_id": "12D3KooWKzWKFojk7A1Hw23dpiQRbLs6HrXFf4EGLsN4oZ1WsWCc",
  "local_server_port": 5001,
  "external_server_port": 5002,
  "psk_file_path": "test/tmp/alice/psk",
  "psk_sig_file_path": "test/tmp/alice/psk_sig",
  "node_key_file_path": "test/tmp/alice/node_key",
  "key_rotation_time": 50,
  "qrng_api_key": "",
  "qkd_cert_path": null,
  "qkd_cert_key_path": null,
  "peers": {
    "12D3KooWT1niMg9KUXFrcrworoNBmF9DTqaswSuDpdX8tBLjAvpW": {
      "qkd_addr": "http://localhost:8182/alice/bob",
      "server_addr": "http://localhost:5004"
    },
    "12D3KooWDNdLiaUM2161yCQMvZy9LVgP3fcySk8nuimcKMDBXryj": {
      "qkd_addr": "http://localhost:8182/alice/dave",
      "server_addr": "http://localhost:5008"
    }
  }
}
```

- **local_peer_id** - local node identifier, which is generated from the private key(node_key) for "peer to peer";
- **local_server_port** - port number for the server, which is responsible for starting the pre-shared key rotation and node reboot process;
- **external_server_port** - port number for the server, which is responsible for transferring the new pre shared key using QKD to the node in need;
- **psk_file_path** - path to the file that contains the pre-shared key;
- **psk_sig_file_path** - path to a file containing the signature of the pre-shared key and the block number when the pre-shared key was generated;
- **node_key_file_path** - path to a file containing the private key fo "peer to peer";
- **key_rotation_time** - the time from the beginning of the pre-shared key rotation at which nodes are allowed ti receive this key (number of milliseconds);
- **qrng_api_key** - key to access the qrng used to generate pre-shared key;
- **peers** - information about the nodes with which we create a connection using QKD. It contains the node identifier, the address of the QKD-simulator and the external server address of the given node;

### 3.1. Using Python

Quantum Meta-chain introduces a concept of **Pre-shared key rotation**.
To make things work we introduced a system for managing rotating those keys called a **runner**.
Runner works as a wrapper around Rust-built code that rotates pre-shared keys after some period of time.
To run a Rust-built node run a following command:

```bash
python3 runner/app.py --config-file <config_path> --process './target/release/qmc-node \
--base-path /tmp/<node_name> \
--chain ./quantumMetachainSpecRaw.json \
--name <node_name> \
--port <port_number> \
--ws-port <port_number> \
--rpc-port <port_number> \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--rpc-methods Unsafe \
--no-mdns'
```

If you want to run a configuration f.e. for alice node, you would need to change configuration to that on of alice:

```bash
python3 runner/app.py --config-file ./path/to/alice/config.json --process './target/release/qmc-node \
--chain ./quantumMetachainSpecRaw.json \
--name Alice \

...
```

For a list of all available flags run your `qmc-node` with `--help` flag

### 3.2. Using Docker

To run a Docker container from docker image built in earlier steps run:

```bash
docker run -it quantum-metachain
```

You can also use "make" to start a network of four nodes:

```bash
make start
# and
make stop
```

## 4. Testing

There are few layers that need to be covered with testing suites:

- Quantum Meta-chain code
- Runner code
- Key rotating flow
  Each of those layers have their separate way of writing/running tests

### 4.1 Runner unit tests

To run runner unit tests:

```bash
pytest ./runner/test --ignore=psk_rotation_test.py
```

### 4.2 Rust unit tests

To run QMC unit tests:

```bash
cargo test
```

### 4.3 Key rotation tests

To run integration key rotation tests:

```bash
cd runner
python3 psk_rotation_test.py
```

## 5. Documentation

To generate documentation run:

```bash
cargo doc
```

## 6. The whitepaper

https://www.quantumblockchains.io/wp-content/uploads/2023/06/QBCK_WhitePaper.pdf

In order to display documentation go to `target/doc/<crate you want to see>` and open `index.html` file in the browser that you want to, e.g.

#### MAC

```bash
cd target/doc/qmc_node
open -a "Google Chrome" index.html
```

#### Linux

```bash
cd target/doc/qmc_node
firefox index.html
```

## Build & Vendoring fixes (developer notes)

This section documents recent local fixes made to get the repository to build locally when dependency conflicts appeared.

- Vendored `schnorrkel` v0.9.1 into `vendor/schnorrkel` and added a patch so workspace consumers resolve the same version.
- Vendored `substrate-bip39` v0.4.6 into `vendor/substrate-bip39-0.4.6` and updated its `Cargo.toml` to depend on the vendored `schnorrkel` (path = `../schnorrkel`).
- Cloned the Quantum-Blockchains Substrate fork (branch `qmc-v0.0.1`) into `vendor/substrate` to allow small local edits and path overrides.
- Patched `vendor/substrate/primitives/io/src/lib.rs` to remove an invalid `#[no_mangle]` on the panic handler (rustc rejects `#[no_mangle]` on internal language items).
- Added `patch` entries to the workspace `Cargo.toml` to force several substrate primitive crates (for example `sp-io`, `sp-core`, `sp-runtime`, `sp-std`, `sp-trie`, `sp-externalities`, `sp-storage`, `sp-runtime-interface`, `sp-state-machine`, `sp-wasm-interface`, `sp-tracing`, `sp-debug-derive`) to use the local `vendor/substrate/primitives/*` copies.

What was observed while iterating:

- Initial cause: two different `schnorrkel` releases (v0.9.1 vs v0.11.x) and `sha2` version divergence produced E0308 type mismatch errors in sr25519-related code.
- After vendoring and patching, builds showed "Patch ... was not used in the crate graph" warnings when the patched source/version/features did not match the dependency requirements.
- A rustc error about `#[no_mangle]` on internal language items appeared because an unpatched git checkout of `sp-io` was still being resolved by Cargo; vendoring `sp-io` and patching it fixed that specific error.
- The build later surfaced duplicate `sp-core` and other `sp-*` crates (some resolved from the git checkout and some from `vendor/`), leading to further E0308 mismatches; the mitigation is to ensure all `sp-*` crates pulled from the substrate git source are overridden to `vendor/substrate/primitives/*` via exact `patch` entries.

Commands used while debugging (run from repo root):

```bash
rm -f Cargo.lock
cargo clean
cargo update
cargo build -p qmc-node --manifest-path bin/node/Cargo.toml -v
```

Next recommended steps (if build still fails):

- Ensure the root `Cargo.toml` contains `patch` entries for every crate that the substrate git source exposes into the graph (add any missing `sp-*` names reported in "Patch ... was not used").
- After editing `Cargo.toml` run `rm -f Cargo.lock && cargo update` to force re-resolution.
- Use `cargo tree -d` to find duplicates and `cargo tree -p schnorrkel` or `cargo tree -p sp-core` to verify which path is used.
- If the repository continues to resolve some substrate crates from the network git checkout (`~/.cargo/git/checkouts/...`), add the exact source patch for `"https://github.com/Quantum-Blockchains/substrate.git"` mapping the crate name to the corresponding `vendor/substrate/primitives/*` path.

If you want, I can continue iterating to make the build green by adding any missing `patch` entries reported by Cargo and re-running the build until there are no duplicate `sp-*` crates remaining.
