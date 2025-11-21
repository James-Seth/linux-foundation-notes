# Chapter 15: Key-Value Pair Stores for Cloud Infrastructure

## Overview

Key-value (KV) stores are essential for distributed, scalable environments. They provide a central source of truth for configuration, service discovery, and coordination. Most KV stores offer REST APIs (GET, PUT, DELETE) for easy integration.

**Popular KV Stores:**
- etcd
- Consul KV
- ZooKeeper

---

## etcd

- **Type:** Open source, distributed KV store
- **Origin:** Started by CoreOS, now part of CNCF
- **Language:** Go
- **Consensus:** Uses Raft algorithm for strong consistency and leader election
- **Deployment:** Standalone or clustered (recommended for high availability)
- **Security:** Supports SSL/TLS, client certificates, and authentication
- **API:** RESTful (HTTP/JSON), supports watch for key changes
- **Performance:** ~10,000 writes/sec per instance (benchmarked)
- **Integration:** Default KV store for Kubernetes, used by CoreDNS, rook, vulcand, OpenStack

**Key Use Cases:**
- Store cluster configuration, bootstrapping keys, and metadata
- Service discovery (with skyDNS, CoreDNS)
- Distributed locking and leader election
- Watch for changes to trigger automated actions

**Example Commands:**
- Set a key:  
  `etcdctl put /app/config/db_host db.example.com`
- Get a key:  
  `etcdctl get /app/config/db_host`
- Watch a key for changes:  
  `etcdctl watch /app/config/db_host`
- List all keys:  
  `etcdctl get "" --prefix --keys-only`

**API Example (HTTP):**
```bash
curl -L http://localhost:2379/v3/kv/put -X POST \
  -d '{"key":"YXBwL2NvbmZpZy9kYl9ob3N0","value":"ZGIuZXhhbXBsZS5jb20="}'
```
(Base64-encoded key and value)

**Real-World Example:**
- Kubernetes stores all cluster state (nodes, pods, secrets, config maps) in etcd. If etcd is unavailable, the cluster cannot function. etcd was chosen for Kubernetes because of its strong consistency and reliable leader election, which are critical for cluster health.
- CoreDNS uses etcd as a backend for dynamic DNS records in container environments, enabling real-time DNS updates as containers come and go.

**Security Best Practices:**
- Always enable TLS for client and peer communication
- Use client certificate authentication
- Restrict etcd API access to trusted networks
- Regularly back up etcd data

**Limitations:**
- Sensitive to network partitions; split-brain can cause data loss
- Write throughput can be limited by consensus protocol
- Not designed for large binary blobs or high-latency WAN replication

**Cross-Reference:**
- See Chapter 14 for Kubernetes architecture and how etcd fits into cluster management.

---

## Consul KV

- **Type:** Distributed, highly-available KV store (by HashiCorp)
- **Consensus:** Uses Raft (via Serf for membership/failure detection)
- **Deployment:** Single or multi-node (multi-node recommended)
- **Security:** Supports ACLs, TLS encryption, and service authentication
- **API:** RESTful (HTTP/JSON), integrates with DNS for service discovery
- **Features:**
  - Service discovery (DNS/HTTP)
  - Health checks for services and nodes
  - Multi-datacenter support
  - Dynamic load balancing and network automation
  - Blue/Green and Canary deployment support

**Key Use Cases:**
- Store configuration and connection info
- Service discovery and health monitoring
- Secure service-to-service communication
- Automate network infrastructure

**Example Commands:**
- Set a key:  
  `consul kv put app/config/db_host db.example.com`
- Get a key:  
  `consul kv get app/config/db_host`
- List all keys:  
  `consul kv export`

**API Example (HTTP):**
```bash
curl --request PUT http://localhost:8500/v1/kv/app/config/db_host \
  --data "db.example.com"
curl http://localhost:8500/v1/kv/app/config/db_host?raw
```

**Real-World Example:**
- In a microservices environment, Service A registers itself in Consul. Service B queries Consul DNS to find Service A’s IP and port, enabling dynamic service discovery. Consul was chosen for its native DNS integration and health checks, which allow for automatic removal of failed nodes and seamless scaling.
- Consul is often used in multi-cloud or hybrid environments to provide a unified service registry and secure service-to-service communication.

**Security Best Practices:**
- Enable ACLs to restrict access to KV data and APIs
- Use TLS for all agent and client communication
- Limit write access to trusted services only
- Regularly audit Consul logs and access patterns

**Limitations:**
- Consul’s Raft consensus can be sensitive to network latency in multi-datacenter setups
- Requires careful ACL management for security
- Not optimized for very large values or high write throughput

**Cross-Reference:**
- See Chapter 14 for Terraform integration with Consul for remote state storage and service discovery.

---

## ZooKeeper

- **Type:** Centralized coordination and KV store (Apache Foundation)
- **Consensus:** ZAB (ZooKeeper Atomic Broadcast) protocol for consistency
- **Deployment:** Clustered for high availability (odd number of nodes recommended)
- **Security:** Supports authentication, ACLs, and TLS encryption
- **API:** Hierarchical namespace (like a filesystem), supports watches and ephemeral nodes
- **Features:**
  - Distributed synchronization and group management
  - Leader election and failover
  - Ordered, atomic updates
  - Backing store for distributed data structures

**Key Use Cases:**
- Node coordination and membership management
- Distributed job scheduling and lock management
- High availability leader election
- Lightweight failover and load balancing

**Example Commands:**
- Create a znode:  
  `zkCli.sh create /app/config/db_host db.example.com`
- Get a znode value:  
  `zkCli.sh get /app/config/db_host`
- List znodes:  
  `zkCli.sh ls /app/config`
- Set a watch:  
  `zkCli.sh get /app/config/db_host true`

**API Example (Java):**
```java
ZooKeeper zk = new ZooKeeper("localhost:2181", 3000, null);
zk.create("/app/config/db_host", "db.example.com".getBytes(),
  Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
byte[] data = zk.getData("/app/config/db_host", false, null);
System.out.println(new String(data));
```

**Real-World Example:**
- Apache Kafka uses ZooKeeper to manage broker metadata, elect controllers, and coordinate partition assignments. ZooKeeper was chosen for its reliable leader election and ability to maintain consistent state across distributed brokers.
- Hadoop YARN uses ZooKeeper for resource manager failover and distributed locking, ensuring high availability in big data clusters.

**Security Best Practices:**
- Always enable TLS for client-server and server-server communication
- Use strong ACLs to restrict access to znodes
- Run ZooKeeper on a private network, not exposed to the public internet
- Monitor and audit access to sensitive znodes

**Limitations:**
- Complex to operate at large scale (requires tuning and monitoring)
- Not designed for high write throughput or large data storage
- Requires odd number of nodes for quorum; split-brain can cause issues

**Cross-Reference:**
- See Chapter 14 for BOSH and Cloud Foundry use of ZooKeeper for distributed coordination.

---

## Comparison Table

| Feature         | etcd                | Consul KV           | ZooKeeper           |
|----------------|---------------------|---------------------|---------------------|
| Consensus      | Raft                | Raft/Serf           | ZAB                 |
| API            | REST (HTTP/JSON)    | REST, DNS           | Custom, REST        |
| Data Model     | Flat KV             | Flat KV             | Hierarchical (tree) |
| Security       | TLS, Auth           | TLS, ACLs           | TLS, ACLs           |
| Use in K8s     | Yes (default)       | Optional            | No                  |
| Use in Mesos   | No                  | No                  | Yes (default)       |
| Service Disc.  | With CoreDNS/skyDNS | Native              | No                  |
| Health Checks  | No                  | Yes                 | No                  |
| Multi-DC       | No                  | Yes                 | No                  |

---

## Glossary

- **Consensus Algorithm:** Protocol to ensure all nodes in a distributed system agree on shared state (e.g., Raft, ZAB).
- **Raft:** A consensus algorithm designed for understandability and strong consistency.
- **ZAB (ZooKeeper Atomic Broadcast):** ZooKeeper’s protocol for ordering updates and maintaining consistency.
- **Ephemeral Node:** Temporary node in ZooKeeper that exists as long as the session is active.
- **Service Discovery:** Mechanism for services to find and communicate with each other dynamically.
- **ACL (Access Control List):** Rules that define permissions for accessing resources.
- **Quorum:** Minimum number of nodes required to make decisions in a distributed system.
- **Leader Election:** Process of selecting a single node as the coordinator among distributed nodes.
- **Split-Brain:** Situation where network partition causes two groups of nodes to operate independently, risking data inconsistency.

---

## Summary

- Key-value stores are critical for distributed system configuration, service discovery, and coordination.
- etcd is the default for Kubernetes, Consul KV is popular for service discovery and multi-datacenter setups, and ZooKeeper is widely used for distributed coordination and leader election.
- Choose based on your platform, consistency needs, and integration requirements.


