# BitStore 📦

![BitStore UI](assets/frontend.png)

> **Decentralized. Encrypted. Permanent.**

BitStore is a distributed, content-addressable object storage system inspired by principles from IPFS and BitTorrent. It splits files into cryptographically hashed blocks, ensuring data deduplication and integrity verification at the byte level.

Designed as a **Microservices Monorepo**, it decouples the storage plane from the metadata plane, allowing for independent scaling of storage nodes versus file orchestration.

---

## 🧠 How It Works (The Core Logic)

BitStore is not just a file uploader; it is a **Content Addressable Storage (CAS)** engine. Here is the lifecycle of a file:

### 1. ✂️ Chunking (The Split)
When a file is uploaded, the **Metadata Service** does not save it as one blob. Instead, it streams the file and slices it into fixed-size **1MB Blocks**.
* *Why?* This allows us to process massive files without eating up RAM, and enables parallel processing.

### 2. #️⃣ Hashing & Integrity
Each 1MB block is passed through a **SHA-256** cryptographic function.
* The output hash (e.g., `a1b2c3...`) becomes the **Unique ID** of that block.
* This ensures data integrity: if a single bit changes, the hash changes, and the system knows the data is corrupted.

### 3. ♻️ Deduplication (The Magic)
Before storing a block, the system checks: *"Do I already have a block with hash `a1b2c3...`?"*
* **Yes:** We discard the incoming data and simply point the new file to the existing block.
* **No:** We write the new block to the **Block Service**.
* *Result:* If 1,000 users upload the exact same 50MB video, BitStore only stores it **once**, saving 99.9% of storage space.

### 4. 🧬 File DNA (The Visualizer)
The frontend features a real-time **Inspector Panel**. When you click any file, you can see its "DNA"—the list of constituent block hashes. This provides visual proof of deduplication: upload the same file twice, and you will see the exact same block hashes being reused.

---

## 🏗 Architecture & Stack

BitStore is fully Dockerized and runs on four orchestrated containers:

### **Metadata Service (The Brain)**
* **Stack:** Java 17, Spring Boot, Hibernate.
* **Role:** Maintains the `File Name -> [List of Block Hashes]` mapping. It never touches the raw data on disk, keeping it lightweight.

### **Block Service (The Vault)**
* **Stack:** Java 17, Spring Boot, Local IO.
* **Role:** A "Dumb" storage node. It doesn't know what "File.jpg" is; it only knows it holds a block named `8f4b...`.

### **PostgreSQL (The Index)**
* **Role:** Relational persistence for file metadata and ownership records.

### **Frontend & Gateway (The Interface)**
* **Stack:** React, Vite, Tailwind CSS, Nginx.
* **Role:** Nginx serves the React UI and acts as a reverse proxy/load balancer for API requests.

---

## 🚀 Getting Started

You can run the entire distributed system locally with a single command.

### Prerequisites
* Docker & Docker Compose installed.

### Installation

1.  **Clone the Repository**
    ```bash
    git clone [https://github.com/Archnightt/BitStore.git](https://github.com/Archnightt/BitStore.git)
    cd BitStore
    ```

2.  **Launch the System**
    ```bash
    docker-compose up --build
    ```
    *(The first build may take a few minutes as it downloads dependencies and compiles the Java images).*

3.  **Access the Cloud**
    Open `http://localhost:5173` in your browser.

---

## 📄 License

**Open Source.**
This project is free to use, modify, and distribute. Built by **Harsh Kolarkar**.