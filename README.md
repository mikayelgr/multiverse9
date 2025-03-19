# Multiverse9 - Distributed Key-Value Storage Network

![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)

Multiverse9 is a distributed key-value storage system that allows multiple nodes to communicate and share data over a TCP-based protocol. It provides a reliable mechanism for creating, removing, and aggregating data across a network of nodes. While the project is in its early stages, it offers a solid foundation for building a distributed storage network.

## Overview

Multiverse9 consists of a core library and a command-line interface for interacting with nodes:

- **multiverse9core**: The core library implementing the node logic, protocol handling, and network communication
- **multiverse9ctl**: A command-line tool to set up and run Multiverse9 nodes

## Features

- **Distributed Architecture**: Connect multiple nodes in a network configuration
- **Redis Backend**: Uses Redis for persistent storage of data
- **Thread Pooling**: Efficiently handles multiple concurrent connections
- **Custom Binary Protocol**: Lightweight communication protocol between nodes
- **Node Permissions**: Configure access control between nodes
- **Auto-generated ULIDs**: Every stored value receives a unique ULID identifier

## Commands

The protocol supports three primary operations:

1. **Create** (0x0001): Store data on a node, generating a unique ULID key
2. **Remove** (0x0002): Delete one or more keys from storage
3. **Aggregate** (0x0003): Retrieve values associated with specified keys, optionally from remote nodes

## Getting Started

### Prerequisites

- Rust toolchain (stable channel)
- Redis server

### Installation

Clone the repository and build the project:

```bash
git clone https://github.com/yourusername/multiverse9.git
cd multiverse9
cargo build --release
```

### Setting Up a Node

Generate a settings file for a new node:

```bash
./target/release/multiverse9ctl setup --redis-uri="redis://127.0.0.1:6379" > instances/settings-1.json
```

Modify the generated settings as needed (e.g., change the binding address).

### Running a Node

Start a node using the settings file:

```bash
./target/release/multiverse9ctl run -s instances/settings-1.json
```

You can also specify the number of worker threads:

```bash
./target/release/multiverse9ctl run -s instances/settings-1.json --threads 8
```

## Configuration

The settings file contains the following main configuration options:

- `name`: Human-readable identifier for the node
- `redis_uri`: Connection string for the Redis server
- `addr`: Binding IP address and port for the node
- `perms`: Permissions for node interactions
- `open_metadata`: Whether the node allows metadata requests
- `open_interactions`: Whether the node allows unrestricted access
- `nodes`: List of acknowledged nodes that can interact with this node

## Query Examples

### Create Data

To create data on a node, use the `CREATE` command:

```bash
echo -n -e "\x00\x01Hello, World!" | nc <node_address> <node_port>
```

### Remove Data

To remove data from a node, use the `REMOVE` command with the ULID of the data:

```bash
echo -n -e "\x00\x02<ULID>" | nc <node_address> <node_port>
```

### Aggregate Data

To aggregate data from a node, use the `AGGREGATE` command with the ULIDs of the data:

```bash
echo -n -e "\x00\x03<ULID1>\x00<ULID2>" | nc <node_address> <node_port>
```

## Development

### Logging

Use the -d or --debug flag to enable debug logging:

```bash
./target/release/multiverse9ctl -d run -s instances/settings-1.json
```

Alternatively, set the RUST_LOG environment variable:

```bash
RUST_LOG=debug ./target/release/multiverse9ctl run -s instances/settings-1.json
```

### Multiple Nodes

You can run multiple nodes on different ports for testing:

```bash
# Terminal 1
./target/release/multiverse9ctl -d run -s instances/settings-1.json

# Terminal 2
./target/release/multiverse9ctl -d run -s instances/settings-2.json
```

### VSCode Launch Configuration

The repository includes VSCode launch configurations for debugging two nodes simultaneously.

## Technical Details

### Thread Pooling

The system uses a custom thread pool implementation to handle incoming connections efficiently. By default, it creates 14 worker threads, but this is configurable when starting a node.

### Network Protocol

The protocol uses a binary format where the first byte indicates the command type, followed by the payload:

- For `CREATE` commands, the payload is stored directly
- For `REMOVE`, keys are separated by NULL bytes (0x0)
- For `AGGREGATE`, keys are separated by NULL bytes and can optionally include a remote node address using the `@` symbol

### ULID Generation

Each data entry is assigned a Universally Unique Lexicographically Sortable Identifier (ULID), providing time-ordered unique identifiers.

## License

This project is licensed under the GNU Affero General Public License v3.0 (AGPL-3.0) - see the [LICENSE](./LICENSE) file for details.
