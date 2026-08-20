# Software Engineering Handbook

> A living engineering handbook exploring software design, architecture, distributed systems, system design, and AI engineering through practical examples, trade-offs, and architectural decisions.

This repository documents my ongoing exploration of software engineering concepts, patterns, architectures, and real-world engineering practices.

The goal is not to create another collection of definitions, but to understand **why** a particular approach works, **when** to use it, **when not to use it**, and what trade-offs it introduces.

---

## 🎯 What This Repository Covers

The handbook is organized around the areas I use and continue to explore as a software engineer and architect.

### 🧱 Software Engineering Principles

Fundamental principles for designing maintainable and extensible software.

- SOLID
- DRY
- KISS
- YAGNI
- Separation of Concerns
- Cohesion & Coupling
- Composition over Inheritance
- Law of Demeter
- Fail Fast
- Make Invalid States Unrepresentable

Each principle focuses on:

- The problem it addresses
- The underlying concept
- Bad vs. improved implementations
- Trade-offs
- When to use it
- When not to use it
- Language-specific considerations

---

### 🧩 Design Patterns

Practical implementations and analysis of common software design patterns.

#### Creational

- Factory
- Abstract Factory
- Builder
- Prototype
- Singleton

#### Structural

- Adapter
- Decorator
- Facade
- Proxy
- Composite

#### Behavioral

- Strategy
- Observer
- Command
- State
- Chain of Responsibility
- Template Method

The focus is not on memorizing patterns, but on understanding:

> **Problem → Context → Pattern → Trade-offs → Alternatives**

---

### 🏗️ Software Architecture

Exploring architectural approaches and their trade-offs.

- Clean Architecture
- Hexagonal Architecture
- Onion Architecture
- Modular Monolith
- Microservices
- Event-Driven Architecture
- CQRS
- Event Sourcing
- Saga Pattern
- Strangler Fig Pattern
- API Gateway

A key focus is understanding:

> **When should we use an architecture, and when should we avoid it?**

---

### 🌐 Distributed Systems

Concepts required to design reliable and scalable distributed systems.

- Scalability
- Availability
- Reliability
- Consistency
- CAP Theorem
- Replication
- Partitioning
- Sharding
- Caching
- Load Balancing
- Messaging
- Queues & Streams
- Idempotency
- Distributed Locks
- Consensus

---

### 🗄️ Databases

Understanding database design and the trade-offs between different storage models.

- Relational Databases
- NoSQL Databases
- Indexing
- Transactions
- Isolation Levels
- Normalization
- Denormalization
- Query Optimization
- Replication
- Partitioning
- Database Selection

---

### 🔌 API Design

Designing APIs that are scalable, maintainable, secure, and easy to evolve.

- REST
- gRPC
- GraphQL
- API Versioning
- Pagination
- Rate Limiting
- Idempotency
- Error Handling
- API Security

---

### ☁️ Cloud & Infrastructure

Exploring cloud-native architecture and infrastructure concepts.

- Compute
- Storage
- Networking
- Containers
- Kubernetes
- Serverless
- Observability
- AWS Architecture
- Cloud Security

---

### 🔐 Security

Security principles and practices for modern software systems.

- Authentication
- Authorization
- OAuth
- JWT
- Encryption
- Secrets Management
- Threat Modeling
- STRIDE
- OWASP
- Secure API Design

---

### 📱 Mobile Engineering

Architecture and engineering practices for mobile applications.

- Mobile Architecture
- Modularization
- Offline-First Architecture
- Data Synchronization
- Networking
- Caching
- Push Notifications

#### iOS

- Swift
- SwiftUI
- Concurrency
- Memory Management
- Dependency Injection
- Modular Architecture

---

### 🌐 Web Engineering

Modern web architecture and development concepts.

- Browser Fundamentals
- JavaScript
- TypeScript
- React
- Frontend Architecture
- State Management
- Web Performance
- Web Security

---

### 🤖 AI Engineering

Exploring modern AI application engineering and LLM-based systems.

- LLM Fundamentals
- Tokenization
- Embeddings
- Vector Databases
- Prompt Engineering
- Structured Output
- Function Calling

#### RAG

- Document Ingestion
- Chunking
- Embeddings
- Retrieval
- Hybrid Search
- Reranking
- Context Engineering
- Evaluation

#### Agentic AI

- Agent Fundamentals
- Tool Calling
- Planning
- Memory
- Reflection
- Routing
- Multi-Agent Systems
- Agent Evaluation
- Agent Observability

---

## 💻 Languages

Examples and implementations will primarily use:

- **Swift**
- **Go**
- **TypeScript**
- **Python**

The goal is not to implement every concept in every language.

Instead, examples will be selected where the language itself provides interesting differences in design, abstraction, type systems, concurrency, or programming paradigms.

For selected concepts such as SOLID and dependency inversion, implementations across multiple languages will be used to demonstrate how the same engineering principle can be expressed differently.

---

## 🧠 How Each Topic Is Explored

Where appropriate, topics follow a consistent structure:

```text
Problem
   ↓
Concept
   ↓
Naive / Problematic Implementation
   ↓
Improved Design
   ↓
Trade-offs
   ↓
When to Use
   ↓
When NOT to Use
   ↓
Language Considerations
   ↓
Real-World Application