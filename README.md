<img src="https://trufflesuite.com/img/truffle-logo-dark.svg" width="200">

[![npm](https://img.shields.io/npm/v/truffle.svg)](https://www.npmjs.com/package/truffle)
[![npm](https://img.shields.io/npm/dm/truffle.svg)](https://www.npmjs.com/package/truffle)
[![Join the chat at https://gitter.im/consensys/truffle](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/consensys/truffle?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Join the community on Spectrum](https://withspectrum.github.io/badge/badge.svg)](https://spectrum.chat/trufflesuite/truffle)
[![Build Status](https://travis-ci.org/trufflesuite/truffle.svg)](https://travis-ci.org/trufflesuite/truffle)
[![Coverage Status](https://coveralls.io/repos/github/trufflesuite/truffle/badge.svg)](https://coveralls.io/github/trufflesuite/truffle)

-----------------------

## Overview

Truffle is a development environment, testing framework and asset pipeline for Ethereum, aiming to make life as an Ethereum developer easier. With Truffle, you get:

* Built-in smart contract compilation, linking, deployment and binary management.
* Automated contract testing with Mocha and Chai.
* Configurable build pipeline with support for custom build processes.
* Scriptable deployment & migrations framework.
* Network management for deploying to many public & private networks.
* Interactive console for direct contract communication.
* Instant rebuilding of assets during development.
* External script runner that executes scripts within a Truffle environment.

| ℹ️ **Contributors**: Please see the [Development](#development) section of this README. |
| --- |

---

## Table of Contents

- [Install](#install)
- [Quick Usage](#quick-usage)
- [Repository Structure](#repository-structure)
  - [Architecture](#architecture)
  - [Package Overview](#package-overview)
    - [Core Framework](#core-framework)
    - [Smart Contract Compilation](#smart-contract-compilation)
    - [Contract Artifacts & Schema](#contract-artifacts--schema)
    - [Deployment & Migrations](#deployment--migrations)
    - [Debugging & Analysis](#debugging--analysis)
    - [Configuration & Environment](#configuration--environment)
    - [Blockchain & Provider Integration](#blockchain--provider-integration)
    - [Provisioning & Module Loading](#provisioning--module-loading)
    - [Testing & Reporting](#testing--reporting)
    - [Data & Storage](#data--storage)
    - [Utilities & Primitives](#utilities--primitives)
  - [Package Dependency Graph](#package-dependency-graph)
- [Development](#development)
  - [Testing Infrastructure](#testing-infrastructure)
  - [CI/CD](#cicd)
- [Documentation](#documentation)
- [License](#license)

---

## Install

```
$ npm install -g truffle
```

## Quick Usage

For a default set of contracts and tests, run the following within an empty project directory:

```
$ truffle init
```

From there, you can run `truffle compile`, `truffle migrate` and `truffle test` to compile your contracts, deploy those contracts to the network, and run their associated unit tests.

Truffle comes bundled with a local development blockchain server that launches automatically when you invoke the commands above. If you'd like to [configure a more advanced development environment](https://trufflesuite.com/docs/advanced/configuration) we recommend you install the blockchain server separately by running `npm install -g ganache-cli` at the command line.

+  [ganache-cli](https://github.com/trufflesuite/ganache-cli): a command-line version of Truffle's blockchain server.
+  [ganache](https://trufflesuite.com/ganache/): A GUI for the server that displays your transaction history and chain state.

---

## Repository Structure

This repository is a **Lerna monorepo** using Yarn workspaces. All packages live under the `packages/` directory and are independently versioned.

```
.
├── packages/               # All individual npm packages
│   ├── abi-utils/
│   ├── artifactor/
│   ├── blockchain-utils/
│   ├── box/
│   ├── code-utils/
│   ├── codec/
│   ├── compile-common/
│   ├── compile-solidity/
│   ├── compile-vyper/
│   ├── config/
│   ├── contract/
│   ├── contract-schema/
│   ├── contract-sources/
│   ├── contract-tests/
│   ├── core/
│   ├── db/
│   ├── debug-utils/
│   ├── debugger/
│   ├── decoder/
│   ├── deployer/
│   ├── environment/
│   ├── error/
│   ├── events/
│   ├── expect/
│   ├── external-compile/
│   ├── hdwallet-provider/
│   ├── interface-adapter/
│   ├── migrate/
│   ├── provider/
│   ├── provisioner/
│   ├── reporters/
│   ├── require/
│   ├── resolver/
│   ├── solidity-utils/
│   ├── source-fetcher/
│   ├── truffle/
│   └── workflow-compile/
├── scripts/                # Release, CI, and utility scripts
├── .github/workflows/      # GitHub Actions CI configuration
├── lerna.json              # Lerna monorepo configuration
└── package.json            # Root workspace configuration
```

### Architecture

The codebase follows a **layered, modular architecture**:

```
┌─────────────────────────────────────────────┐
│           @truffle/truffle (CLI)            │  Distribution layer
├─────────────────────────────────────────────┤
│             @truffle/core                   │  Orchestration layer
├──────────┬──────────┬──────────┬────────────┤
│ Compile  │  Deploy  │  Debug   │  Test      │  Feature layer
├──────────┴──────────┴──────────┴────────────┤
│  config · provider · resolver · contract    │  Integration layer
├─────────────────────────────────────────────┤
│  codec · abi-utils · solidity-utils · error │  Utility layer
└─────────────────────────────────────────────┘
```

### Package Overview

#### Core Framework

| Package | Description |
|---------|-------------|
| **[@truffle/core](packages/core)** | Main Truffle CLI and command orchestration. Implements all user-facing commands: `compile`, `test`, `migrate`, `debug`, `develop`, `console`, `exec`, `unbox`, `watch`, `networks`, and `version`. |
| **[@truffle/truffle](packages/truffle)** | Distribution package that bundles the CLI into a single webpack artifact. This is the `truffle` package published to npm. |

#### Smart Contract Compilation

| Package | Description |
|---------|-------------|
| **[@truffle/compile-solidity](packages/compile-solidity)** | Compiles `.sol` Solidity source files using the `solc` compiler. Manages compiler selection, input/output handling, and artifact generation. |
| **[@truffle/compile-vyper](packages/compile-vyper)** | Compiles Vyper (`.vy`) smart contracts using the `vyper` compiler. |
| **[@truffle/compile-common](packages/compile-common)** | Shared infrastructure for all compiler integrations: common types, a file profiler (import resolution, dependency graph), and shims for legacy/new compiler artifact formats. |
| **[@truffle/external-compile](packages/external-compile)** | Supports arbitrary shell commands as compilation steps, allowing integration of any build tool. |
| **[@truffle/workflow-compile](packages/workflow-compile)** | Orchestrates the full compilation workflow: reads configuration, selects compilers, writes artifacts. |
| **[@truffle/resolver](packages/resolver)** | Resolves contract import paths from npm packages, the local file system, and Truffle-specific locations. |
| **[@truffle/contract-sources](packages/contract-sources)** | Discovers all Solidity and Vyper source files in a project directory using glob patterns. |

#### Contract Artifacts & Schema

| Package | Description |
|---------|-------------|
| **[@truffle/contract-schema](packages/contract-schema)** | Defines and validates the formal JSON schema for Truffle contract artifact objects (ABI, bytecode, networks, etc.) using AJV. |
| **[@truffle/artifactor](packages/artifactor)** | Persists contract artifact objects to `.json` files and loads them back. Used after compilation to save build outputs. |
| **[@truffle/contract](packages/contract)** | High-level JavaScript abstraction over deployed contract instances. Provides promise-based transactions, event subscriptions, default values, and library linking. |
| **[@truffle/abi-utils](packages/abi-utils)** | Normalizes and manipulates ABI definitions across Solidity compiler versions. Includes TypeScript types and property-based testing utilities. |

#### Deployment & Migrations

| Package | Description |
|---------|-------------|
| **[@truffle/deployer](packages/deployer)** | Handles contract deployment: sends deployment transactions, manages constructor arguments, links libraries, and optionally registers contracts with ENS. |
| **[@truffle/migrate](packages/migrate)** | Executes migration scripts in order, tracking which migrations have already been run on-chain via the `Migrations` contract. |

#### Debugging & Analysis

| Package | Description |
|---------|-------------|
| **[@truffle/debugger](packages/debugger)** | Full-featured Solidity debugger. Supports stepping through transactions, setting breakpoints, inspecting variables and the call stack, and evaluating watch expressions. Uses a Redux-based state machine internally. |
| **[@truffle/debug-utils](packages/debug-utils)** | Utilities used by the debugger: syntax-highlighted source printing (via Chromafi), address formatting, and output helpers. |
| **[@truffle/codec](packages/codec)** | Low-level encoding and decoding library for all Solidity types. Handles ABI encoding, storage layout decoding, and memory/calldata decoding without requiring a live node. |
| **[@truffle/decoder](packages/decoder)** | High-level user-facing decoder that wraps `@truffle/codec`. Decodes contract state, return values, events, and call data. |
| **[@truffle/solidity-utils](packages/solidity-utils)** | Utilities for working with compiled Solidity: AST traversal, source mapping, bytecode analysis. |
| **[@truffle/code-utils](packages/code-utils)** | Parses raw EVM bytecode into structured instructions and provides disassembly utilities. |

#### Configuration & Environment

| Package | Description |
|---------|-------------|
| **[@truffle/config](packages/config)** | Reads and validates `truffle-config.js`. Manages network definitions, compiler settings, paths, and plugin configuration. |
| **[@truffle/environment](packages/environment)** | Detects and sets up the runtime environment: connects to the configured network, instantiates the provider, and assigns network properties to the config object. |

#### Blockchain & Provider Integration

| Package | Description |
|---------|-------------|
| **[@truffle/provider](packages/provider)** | Utility wrapper around Web3 providers. Normalizes provider creation and adds error handling. |
| **[@truffle/interface-adapter](packages/interface-adapter)** | Network abstraction layer that presents a uniform API over Web3.js and Ethers.js providers, including Ganache support. |
| **[@truffle/hdwallet-provider](packages/hdwallet-provider)** | BIP39/BIP44 HD wallet provider. Derives accounts from a mnemonic or private keys and signs transactions locally before forwarding them. |
| **[@truffle/blockchain-utils](packages/blockchain-utils)** | Helpers for identifying networks (mainnet, testnets) and inspecting blockchain state. |

#### Provisioning & Module Loading

| Package | Description |
|---------|-------------|
| **[@truffle/provisioner](packages/provisioner)** | Provisions contract abstractions from stored artifacts for use inside migration scripts and tests. |
| **[@truffle/require](packages/require)** | Executes external JavaScript files (migration scripts, `truffle exec` scripts) with the full Truffle environment injected. |
| **[@truffle/box](packages/box)** | Downloads and unpacks Truffle Box project templates from GitHub or a local path, and runs post-unpack lifecycle hooks. |

#### Testing & Reporting

| Package | Description |
|---------|-------------|
| **[@truffle/reporters](packages/reporters)** | Formats and displays output for test runs and migration deployments, including spinners, emoji, and structured log lines. |
| **[@truffle/expect](packages/expect)** | Validates that required configuration options are present before a command runs. |
| **[@truffle/contract-tests](packages/contract-tests)** | Internal test suite for `@truffle/contract`. Not published to npm. |

#### Data & Storage

| Package | Description |
|---------|-------------|
| **[@truffle/db](packages/db)** | Aggregates and stores smart contract metadata (sources, bytecodes, compilations, deployments) via a GraphQL API backed by PouchDB. Enables rich cross-project querying of contract data. |

#### Utilities & Primitives

| Package | Description |
|---------|-------------|
| **[@truffle/error](packages/error)** | Extends the native `Error` class with additional properties used consistently across all Truffle packages. |
| **[@truffle/events](packages/events)** | Provides an event emitter (Emittery) and progress-spinner (ora) integration used for inter-package communication and CLI feedback. |
| **[@truffle/source-fetcher](packages/source-fetcher)** | Fetches verified smart contract source code from Etherscan and Sourcify, used by the debugger to obtain sources for contracts not in the local project. |

---

### Package Dependency Graph

The packages form a clear layered dependency hierarchy:

```
@truffle/truffle
└── @truffle/core
    ├── @truffle/workflow-compile
    │   ├── @truffle/compile-solidity
    │   ├── @truffle/compile-vyper
    │   ├── @truffle/compile-common
    │   ├── @truffle/external-compile
    │   └── @truffle/resolver
    ├── @truffle/migrate
    │   └── @truffle/deployer
    │       └── @truffle/contract
    │           └── @truffle/contract-schema
    ├── @truffle/debugger
    │   ├── @truffle/codec
    │   ├── @truffle/solidity-utils
    │   └── @truffle/abi-utils
    ├── @truffle/decoder
    │   └── @truffle/codec
    ├── @truffle/config
    │   └── @truffle/provider
    │       └── @truffle/interface-adapter
    ├── @truffle/environment
    │   └── @truffle/interface-adapter
    └── @truffle/box

Cross-cutting (used by many packages):
  @truffle/error · @truffle/expect · @truffle/events
  @truffle/artifactor · @truffle/contract-schema
```

---

## Development

We welcome pull requests. To get started, just fork this repo, clone it locally, and run:

```shell
# Install
npm install -g yarn
yarn bootstrap

# Test
yarn test

# Adding dependencies to a package
cd packages/<truffle-package>
yarn add <npm-package> [--dev] # Use yarn
```

If you'd like to update a dependency to the same version across all packages, you might find [this utility](https://www.npmjs.com/package/lerna-update-wizard) helpful.

*Notes on project branches:*
+    `master`: Stable, released version (v5)
+    `beta`: Released beta version
+    `develop`: Work targeting stable release (v5)
+    `next`: Upcoming feature development and most new work

Please make pull requests against `next` for any substantial changes. Small changes and bugfixes can be considered for `develop`.

There is a bit more information in the [CONTRIBUTING.md](./CONTRIBUTING.md) file.

### Testing Infrastructure

Tests are managed at the package level. The following tools are used:

| Tool | Purpose |
|------|---------|
| **Mocha** | Primary test runner across all JavaScript packages |
| **Chai** | Assertion library used with Mocha |
| **Jest** | Used in `@truffle/db` for TypeScript-first testing |
| **Sinon** | Mocking and stubbing library |
| **ganache-core** | In-memory Ethereum node for integration tests |
| **NYC** | Code coverage instrumentation and reporting |

Run all tests:

```shell
yarn test
```

Run tests for a single package:

```shell
cd packages/<package-name>
yarn test
```

### CI/CD

**GitHub Actions** (`.github/workflows/nodejs.yml`) runs on all pushes to `master`, `develop`, `next`, and `truffle-db`, as well as all pull requests.

The matrix tests against **Node.js 10, 12, and 14** and runs three modes:

| Mode | Environment Variable | Description |
|------|---------------------|-------------|
| Package tests | `PACKAGES=true` | Runs each package's own test suite |
| Integration tests | `INTEGRATION=true` | Runs cross-package integration tests |
| Geth tests | `GETH=true` | Runs tests against a live Geth node |

**Travis CI** (`.travis.yml`) provides additional coverage reporting via Coveralls.

**Pre-commit hooks** (Husky + lint-staged) automatically run Prettier and ESLint on all staged `.js` and `.ts` files before every commit.

---

## Documentation

Please see the [Official Truffle Documentation](https://trufflesuite.com/docs/) for guides, tips, and examples.

---

## License

MIT
