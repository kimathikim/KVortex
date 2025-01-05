# **System Architecture for KVortex**

**Table of Contents**

- [4. High-Level Overview](#4-high-level-overview)
- [5. Components](#5-components)
    - [5.1. Client Layer](#51-client-layer)
    - [5.2. Node Layer](#52-node-layer)
    - [5.3. Data Sharding](#53-data-sharding)
    - [5.4. Replication](#54-replication)
    - [5.5. Storage Backends](#55-storage-backends)
    - [5.6. Monitoring Dashboard](#56-monitoring-dashboard)
- [6. Data Flow](#6-data-flow)
    - [6.1. Write Operation](#61-write-operation)
    - [6.2. Read Operation](#62-read-operation)
    - [6.3. Delete Operation](#63-delete-operation)
- [7. Fault Tolerance](#7-fault-tolerance)
    - [7.1. Node Failure](#71-node-failure)
    - [7.2. Network Partition](#72-network-partition)
- [8. Performance Optimization](#8-performance-optimization)
    - [8.1. Caching](#81-caching)
    - [7.2. Batching](#72-batching)
    - [7.3. Concurrency](#73-concurrency)
- [8. Deployment](#8-deployment)
    - [8.1. Single Node](#81-single-node)
    - [8.2. Multi-Node Cluster](#82-multi-node-cluster)
- [9. Technology Stack](#9-technology-stack)
    - [9.1. Core Technologies](#91-core-technologies)
    - [9.2. Additional Tools](#92-additional-tools)
- [10. Diagrams](#10-diagrams)
    - [10.1. High-Level Architecture](#101-high-level-architecture)
    - [10.2. Data Flow for Write Operation](#102-data-flow-for-write-operation)

---

### **4. High-Level Overview**
KVortex is a **distributed key-value store** that uses a **peer-to-peer architecture** with **consistent hashing** for data sharding and **replication** for fault tolerance. The system is designed to be **horizontally scalable**, meaning you can add more nodes to handle increased load.

---

### **5. Components**

#### **5.1. Client Layer**
- **RESTful API:**  
  - Provides an HTTP interface for clients to interact with the system.  
  - Supports CRUD operations (Create, Read, Update, Delete) for key-value pairs.  
  - Example Endpoints:  
    - `POST /set` - Store a key-value pair.  
    - `GET /get` - Retrieve a value by key.  
    - `DELETE /delete` - Remove a key-value pair.  

- **Client Libraries:**  
  - Optional libraries in Go, Python, or JavaScript to simplify interaction with the RESTful API.

---

#### **5.2. Node Layer**
Each node in the cluster is responsible for:
- **Storing Data:**  
  - Maintains a portion of the key-value data based on consistent hashing.  
- **Handling Requests:**  
  - Processes client requests and routes them to the appropriate node.  
- **Replication:**  
  - Replicates data to other nodes to ensure fault tolerance.  

---

#### **5.3. Data Sharding**
- **Consistent Hashing:**  
  - Keys are hashed and mapped to nodes using a **hash ring**.  
  - Ensures even distribution of data and minimizes rehashing when nodes are added or removed.  
- **Virtual Nodes:**  
  - Each physical node is represented by multiple virtual nodes on the hash ring for better load balancing.  

---

#### **5.4. Replication**
- **Primary-Replica Model:**  
  - Each key-value pair is stored on a **primary node** and replicated to **N replica nodes** (configurable).  
- **Quorum-Based Writes:**  
  - Ensures data consistency by requiring a majority of replicas to acknowledge writes.  
- **Automatic Failover:**  
  - If a primary node fails, one of the replicas is promoted to primary.  

---

#### **5.5. Storage Backends**
KVortex supports multiple storage backends for flexibility:
- **In-Memory Storage:**  
  - Fast but volatile (data is lost on node restart).  
- **File-Based Storage:**  
  - Persists data to disk for durability.  
- **SQLite Storage:**  
  - Lightweight, disk-based storage with ACID compliance.  

---

#### **5.6. Monitoring Dashboard**
- **Real-Time Metrics:**  
  - Displays cluster health, node status, and performance metrics (e.g., latency, throughput).  
- **Alerting:**  
  - Sends alerts for node failures, high latency, or other critical events.  
- **Visualization:**  
  - Uses charts and graphs to provide insights into system performance.  

---

### **6. Data Flow**

#### **6.1. Write Operation**
4. Client sends a `POST /set` request to the RESTful API.  
5. API hashes the key and determines the primary node using the hash ring.  
6. Primary node stores the key-value pair and replicates it to replica nodes.  
7. Once a quorum of replicas acknowledges the write, the primary node responds to the client.  

#### **6.2. Read Operation**
4. Client sends a `GET /get` request to the RESTful API.  
5. API hashes the key and determines the primary node using the hash ring.  
6. Primary node retrieves the value and returns it to the client.  

#### **6.3. Delete Operation**
4. Client sends a `DELETE /delete` request to the RESTful API.  
5. API hashes the key and determines the primary node using the hash ring.  
6. Primary node deletes the key-value pair and propagates the deletion to replica nodes.  

---

### **7. Fault Tolerance**

#### **7.1. Node Failure**
- **Detection:**  
  - Nodes periodically send heartbeats to each other.  
  - If a node fails to respond, it is marked as offline.  
- **Recovery:**  
  - Replica nodes take over for the failed node.  
  - Data is re-replicated to maintain the desired replication factor.  

#### **7.2. Network Partition**
- **Quorum-Based Operations:**  
  - Ensures consistency by requiring a majority of nodes to agree on writes.  
- **Conflict Resolution:**  
  - Uses timestamps or version vectors to resolve conflicts during network partitions.  

---

### **8. Performance Optimization**

#### **8.1. Caching**
- **In-Memory Cache:**  
  - Frequently accessed keys are cached in memory for faster reads.  
- **LRU Eviction:**  
  - Least Recently Used (LRU) policy to manage cache size.  

#### **7.2. Batching**
- **Write Batching:**  
  - Combines multiple writes into a single operation to reduce I/O overhead.  
- **Read Batching:**  
  - Combines multiple reads into a single operation to reduce network latency.  

#### **7.3. Concurrency**
- **Goroutines:**  
  - Uses Go’s lightweight threads to handle multiple requests concurrently.  
- **Channels:**  
  - Facilitates communication between goroutines for coordination.  

---

### **8. Deployment**

#### **8.1. Single Node**
- For development or testing, KVortex can run on a single node with in-memory storage.

#### **8.2. Multi-Node Cluster**
- For production, deploy KVortex across multiple nodes using Docker and Kubernetes.  
- Use a **service discovery** tool like Consul or etcd to manage node membership.  

---

### **9. Technology Stack**

#### **9.1. Core Technologies**
- **Programming Language:** Go (for concurrency and performance).  
- **Web Framework:** Gin (for the RESTful API).  
- **Storage Backends:** In-memory, file-based, SQLite.  

#### **9.2. Additional Tools**
- **Monitoring:** Prometheus and Grafana for real-time metrics.  
- **Containerization:** Docker for packaging and Kubernetes for orchestration.  
- **Service Discovery:** Consul or etcd for node membership and health checks.  

---

### **10. Diagrams**

#### **10.1. High-Level Architecture**
```plaintext
+-------------------+       +-------------------+       +-------------------+
|    Client App     |       |    Client App     |       |    Client App     |
+-------------------+       +-------------------+       +-------------------+
           |                         |                         |
           |                         |                         |
           v                         v                         v
+---------------------------------------------------------------+
|                        RESTful API Gateway                    |
+---------------------------------------------------------------+
           |                         |                         |
           |                         |                         |
           v                         v                         v
+-------------------+       +-------------------+       +-------------------+
|   Node 3 (Primary)|       |   Node 2 (Replica)|       |   Node 3 (Replica)|
+-------------------+       +-------------------+       +-------------------+
```

#### **10.2. Data Flow for Write Operation**
```plaintext
Client -> REST API -> Hash Ring -> Primary Node -> Replica Nodes -> Client
```

