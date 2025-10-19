# CacheMesh

[![Go Report Card](https://goreportcard.com/badge/github.com/your-repo/cachemesh)](https://goreportcard.com/report/github.com/your-repo/cachemesh)
[![GoDoc](https://godoc.org/github.com/your-repo/cachemesh?status.svg)](https://godoc.org/github.com/your-repo/cachemesh)
[![Build Status](https://travis-ci.org/your-repo/cachemesh.svg?branch=main)](https://travis-ci.org/your-repo/cachemesh)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

**Using P2P Networking For A Decentralized In-Memory Cache Between Microservices**

CacheMesh is a decentralized, distributed in-memory caching system designed specifically for microservices environments. It leverages a peer-to-peer (P2P) network, allowing services to form a caching cluster without a centralized server. This eliminates single points of failure and simplifies deployment.

---

## 💡 What is CacheMesh?

In a typical microservices architecture, services often rely on a centralized caching layer like Redis or Memcached. While powerful, this introduces a separate component to manage, scale, and maintain.

CacheMesh takes a different approach. It embeds a lightweight caching node directly within your application. These nodes discover each other and form a distributed hash table (DHT) across the network. When you write data to the cache on one service, it's distributed to the appropriate peer based on the key. When you read data, your service automatically fetches it from the correct peer if it's not available locally.

This creates a shared, high-speed, in-memory cache that scales horizontally with your services.

## ✨ Features

- **Decentralized:** No central server or coordinator. No single point of failure.
- **Peer-to-Peer Discovery:** Nodes automatically discover each other using a gossip protocol.
- **Consistent Hashing:** Keys are distributed across the cluster using a consistent hashing algorithm, minimizing rebalancing when nodes join or leave.
- **Embeddable:** Integrates directly into your Go application as a library.
- **Lightweight:** Designed to have a small memory and CPU footprint.
- **Scalable:** Scales horizontally as you add more service instances.
- **Simple API:** A familiar `Get`, `Set`, `Delete` API for caching.

## 🏗️ Architecture

Each instance of your microservice embeds a CacheMesh node.

1.  **Bootstrap:** On startup, a node connects to one or more initial "bootstrap" peers to join the cluster.
2.  **Gossip:** Once connected, it uses a gossip protocol to learn about other nodes in the network and maintain an up-to-date membership list.
3.  **Consistent Hashing:** A consistent hash ring is used to map cache keys to specific nodes. This ensures that keys are evenly distributed and that adding/removing nodes has minimal impact on key location.
4.  **Data Flow:**
    -   When you call `Set(key, value)`, the local node hashes the `key` to determine which node in the cluster is responsible for it. It then sends a request to that node to store the data.
    -   When you call `Get(key)`, the local node hashes the `key` and requests the data from the responsible peer.

```
  +------------------+           +------------------+
  |                  |           |                  |
  |   Microservice A |<--------->|   Microservice B |
  | [CacheMesh Node] | Gossip    | [CacheMesh Node] |
  |                  | Protocol  |                  |
  +------------------+           +------------------+
          ^                              ^
          |                              |
          v                              v
  +------------------+           +------------------+
  |                  |           |                  |
  |   Microservice C |<--------->|   Microservice D |
  | [CacheMesh Node] |           | [CacheMesh Node] |
  |                  |           |                  |
  +------------------+           +------------------+

       Consistent Hashing Ring determines data ownership
```

## 🚀 Getting Started

### Prerequisites
- Go 1.18 or later

### Installation

```sh
go get github.com/your-repo/cachemesh
```

### Quick Start

Here is a simple example of how to run two CacheMesh nodes that connect to each other.

**`main.go`**
```go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/your-repo/cachemesh"
)

func main() {
	// Get node port from command-line argument, e.g., ":3000"
	// Get bootstrap node from command-line argument, e.g., "localhost:3000"
	// For simplicity, we'll hardcode them here.

	// Node 1 configuration
	opts1 := cachemesh.Options{
		ListenAddr: ":3000",
		BootstrapNodes: []string{}, // First node has no one to connect to initially
	}

	// Node 2 configuration
	opts2 := cachemesh.Options{
		ListenAddr: ":4000",
		BootstrapNodes: []string{":3000"}, // Second node connects to the first
	}

	// In a real app, you'd run these in separate processes.
	// We use goroutines here to simulate it.
	node1, err := cachemesh.New(opts1)
	if err != nil {
		log.Fatalf("failed to create node 1: %v", err)
	}
	go node1.Start()

	node2, err := cachemesh.New(opts2)
	if err != nil {
		log.Fatalf("failed to create node 2: %v", err)
	}
	go node2.Start()

	// Let the nodes discover each other
	time.Sleep(2 * time.Second)

	// Set a key from Node 2
	key := []byte("hello")
	value := []byte("world")
	ttl := 10 * time.Second

	if err := node2.Set(key, value, ttl); err != nil {
		log.Fatalf("failed to set key: %v", err)
	}
	fmt.Println("Set 'hello' -> 'world' from Node 2")

	// Get the key from Node 1
	retrievedValue, err := node1.Get(key)
	if err != nil {
		log.Fatalf("failed to get key: %v", err)
	}

	fmt.Printf("Got '%s' -> '%s' from Node 1\n", key, retrievedValue)
}
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a pull request.

1.  Fork the repository.
2.  Create your feature branch (`git checkout -b feature/my-new-feature`).
3.  Commit your changes (`git commit -am 'Add some feature'`).
4.  Push to the branch (`git push origin feature/my-new-feature`).
5.  Create a new Pull Request.

## 📜 License

Copyright (c) 2024 Your Name or Company

This project is licensed under the Apache License 2.0 - see the `LICENSE` file for details.
