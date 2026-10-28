# UPI Offline Mesh — Offline Payment & Deferred Settlement System

A **Spring Boot-based simulation of offline UPI payments** using Bluetooth-style mesh routing and deferred settlement. The system demonstrates how encrypted payment packets can travel through a network of virtual devices and reach a backend when an internet-connected bridge becomes available.

Built with Java and Spring Boot, the project explores hybrid encryption, duplicate-payment prevention, transactional settlement, replay protection, and concurrent request handling.

> **Note:** This is an educational simulation. Mesh communication is simulated in software; the project does not implement real Bluetooth communication or integrate with production UPI infrastructure.

## 🚀 Key Features

- **Offline Mesh Simulation:** Routes payment packets across five virtual devices using a Bluetooth-style gossip mechanism.
- **Hybrid Encryption:** Uses RSA-OAEP and AES-256-GCM to protect payment payloads and detect tampering.
- **Idempotent Settlement:** Uses SHA-256 ciphertext hashes and atomic operations to prevent duplicate processing of the same payment.
- **Replay Protection:** Validates payment timestamps and rejects stale packets.
- **Transactional Processing:** Uses Spring transactions to coordinate sender debits, receiver credits, and ledger updates.
- **Concurrency Safety:** Uses atomic operations and optimistic locking to help prevent duplicate settlements and conflicting account updates.
- **REST APIs:** Provides endpoints for payment simulation, mesh operations, bridge ingestion, and transaction tracking.
- **Interactive Dashboard:** Visualizes payment packets, virtual devices, account balances, and settlement outcomes.
- **Automated Tests:** Includes tests for encryption, tampering detection, and concurrent duplicate delivery.

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Java 17 | Core application development |
| Spring Boot | Backend application and REST APIs |
| Spring Data JPA | Database persistence |
| H2 Database | Lightweight in-memory database |
| RSA-OAEP | Encryption of AES session keys |
| AES-256-GCM | Payload encryption and tamper detection |
| SHA-256 | Payment-packet hashing for idempotency |
| Maven | Dependency management and build automation |
| HTML | Interactive demonstration dashboard |
| JUnit | Automated testing |

## 🏗️ System Architecture

The application simulates an offline payment moving through a mesh network before reaching the backend.

```text
Sender
  |
  | Create payment instruction
  v
Hybrid Encryption
(RSA-OAEP + AES-256-GCM)
  |
  v
Virtual Mesh Network
  |
  | Packet forwarding between devices
  v
Internet-Connected Bridge
  |
  | Upload encrypted packet
  v
Spring Boot Backend
  |
  v
SHA-256 Hash & Idempotency Check
  |
  v
Decrypt & Validate Packet
  |
  v
Transactional Settlement
  |
  +--> Debit Sender
  +--> Credit Receiver
  +--> Record Transaction
```

### How the system works

1. **Create a payment:** The application generates a payment instruction containing transaction details, a nonce, and a timestamp.
2. **Encrypt the payment:** The payload is encrypted using AES-256-GCM, while RSA-OAEP protects the AES key.
3. **Route the packet:** The encrypted packet moves between virtual devices through simulated mesh gossip rounds.
4. **Upload through a bridge:** A simulated internet-connected device forwards the packet to the backend.
5. **Check for duplicates:** The backend calculates a SHA-256 hash of the ciphertext and checks whether that packet has already been claimed.
6. **Validate the packet:** The backend decrypts the payload and checks its integrity and freshness.
7. **Settle the payment:** A transactional operation debits the sender, credits the receiver, and records the transaction.

## 🔐 Security and Concurrency

### 1. Hybrid Encryption

The project combines two cryptographic techniques:

- **RSA-OAEP:** Protects the AES session key.
- **AES-256-GCM:** Encrypts the payment payload and authenticates its integrity.

This approach allows intermediate devices to forward encrypted packets without directly reading the payment details. Tampered ciphertext is rejected when authentication fails.

### 2. Idempotent Payment Processing

A payment packet may reach the backend more than once. Processing every delivery could result in duplicate debits.

To address this, the application:

- Calculates a SHA-256 hash of the ciphertext.
- Uses an atomic `ConcurrentHashMap.putIfAbsent()` operation to claim the packet.
- Rejects duplicate deliveries before continuing with settlement.
- Uses a unique database constraint on the packet hash as an additional safeguard.

### 3. Replay Protection

The backend validates the payment timestamp and rejects packets older than the configured freshness window. A nonce distinguishes separate payment instructions, while idempotency checks help prevent the same packet from being settled repeatedly.

### 4. Transactional Settlement

The settlement service uses Spring's `@Transactional` mechanism to coordinate account balance updates and transaction recording. JPA optimistic locking with `@Version` provides an additional safeguard against conflicting updates.

## 📋 Prerequisites

Before running the project, install:

- Java Development Kit (JDK) 17 or newer
- Git
- An internet connection for the initial Maven dependency download

**No separate Maven installation is required** because the repository includes the Maven Wrapper.

## ⚙️ Installation and Setup

### 1. Clone the repository

```bash
git clone https://github.com/Ganesh7sureban/Upi_Without_Internet.git
```

### 2. Navigate to the project directory

```bash
cd Upi_Without_Internet
```

### 3. Run the application on Windows

```powershell
.\mvnw.cmd spring-boot:run
```

On macOS or Linux:

```bash
./mvnw spring-boot:run
```

### 4. Open the dashboard

Once the application starts, visit:

**http://localhost:8080**

The dashboard lets you simulate payment creation, mesh gossip rounds, bridge uploads, and settlement.

## 🧪 Running Tests

Run the complete test suite on Windows:

```powershell
.\mvnw.cmd test
```

On macOS or Linux:

```bash
./mvnw test
```

The documented tests cover:

- Encryption and decryption.
- Rejection of tampered ciphertext.
- Concurrent delivery of the same payment packet.

The concurrency test checks that three simultaneous deliveries of the same packet produce exactly one successful settlement and two duplicate rejections.

## 🔌 API Reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Opens the demonstration dashboard |
| `GET` | `/api/server-key` | Retrieves the server's public key |
| `GET` | `/api/accounts` | Retrieves account balances |
| `GET` | `/api/transactions` | Retrieves recent transactions |
| `GET` | `/api/mesh/state` | Retrieves virtual mesh device states |
| `POST` | `/api/demo/send` | Creates and injects a simulated payment |
| `POST` | `/api/mesh/gossip` | Runs a mesh gossip round |
| `POST` | `/api/mesh/flush` | Uploads packets through bridge devices |
| `POST` | `/api/mesh/reset` | Resets the mesh and idempotency cache |
| `POST` | `/api/bridge/ingest` | Accepts an encrypted packet from a bridge |
| `GET` | `/h2-console` | Opens the H2 database console, if enabled |

Refer to the application controllers and configuration for the exact request formats and response structures.

## 🎯 What I Learned

This project provided practical experience with:

- Designing backend services using Java and Spring Boot.
- Building REST APIs and integrating Spring Data JPA.
- Applying hybrid encryption to application-level data.
- Implementing idempotent operations for transaction processing.
- Understanding race conditions and atomic operations in concurrent systems.
- Managing database updates using transactions and optimistic locking.
- Writing tests for security-related behavior and concurrent requests.
- Modeling distributed-system challenges through a software simulation.

## ⚠️ Limitations

This project is a demonstration, not a production payment system.

- **Simulated networking:** The mesh uses virtual devices; real Bluetooth or Wi-Fi Direct communication is not implemented.
- **Deferred settlement:** A payment is not confirmed as settled until the backend processes it.
- **In-memory database:** H2 is used for demonstration rather than a production database.
- **Local idempotency cache:** `ConcurrentHashMap` is limited to one application instance and does not provide distributed coordination.
- **Production security:** Real deployments would require authentication, authorization, secure key management, monitoring, and appropriate financial-system integration.
- **Offline double-spending:** The simulation does not fully solve the problem of preventing a sender from creating multiple conflicting payments while disconnected.

## 🔮 Potential Improvements

- Integrate a persistent database such as PostgreSQL.
- Use Redis for distributed idempotency across multiple backend instances.
- Add authentication, authorization, and rate limiting.
- Implement secure key management using a dedicated key-management service.
- Explore real device-to-device communication.
- Add observability, structured logging, and performance benchmarks.
- Expand testing for failures, retries, and concurrent account updates.

## 👨‍💻 Author

**Ganesh Muralidhar Sureban**

Computer Science Undergraduate | Backend Development | Java & Spring Boot

- GitHub: [Ganesh7sureban](https://github.com/Ganesh7sureban)

## 📄 License

No license is currently specified for this repository. Add a license file if you intend to grant others explicit permission to use, modify, or distribute the project.
