# Plagiarism-Proof Notes Hub Architecture

## Overview
Plagiarism-Proof Notes Hub is a multi-platform application that combines offline-first clients with a scalable cloud platform to transform handwritten and digital notes into unique, high-quality study material. The system orchestrates OCR pipelines, AI-driven rewriting, plagiarism detection, secure storage, and collaborative tooling through modular services that can be independently deployed and scaled.

The platform targets Android, iOS, Windows, macOS, Linux, and web clients. All clients share a common synchronization protocol and reuse business logic packaged in a cross-platform core written in Kotlin Multiplatform and Rust. Server-side services expose gRPC and REST APIs protected by OAuth 2.1 and rely on event-driven communication through Apache Kafka.

## High-Level Component Map

```
+-------------------+          +--------------------+          +------------------+
|  Client Apps      |  HTTPS   |  API Gateway       |  gRPC    |  Microservices   |
|                   +--------->+  (Envoy)           +--------->+  (Kubernetes)    |
| • Android/iOS     |          |                    |          | • OCR Service    |
| • Desktop (Tauri) |          |                    |          | • AI Rewrite     |
| • Web (Next.js)   |          |                    |          | • Plagiarism     |
+-------------------+          +--------------------+          | • Storage        |
                                                              | • Collaboration  |
                                                              +------------------+
                                                                    |
                                                                    |  Event Bus
                                                                    v
                                                              +------------------+
                                                              |  Data Layer      |
                                                              |  (Postgres, S3,  |
                                                              |   Redis, Neo4j)  |
                                                              +------------------+
```

## Client Architecture

### Shared Core
- **Kotlin Multiplatform SDK**: Business rules, synchronization logic, encryption, offline cache management, and domain models shared across Android, iOS, desktop, and web clients.
- **Rust Native Modules**: Performance-critical features such as on-device OCR (via Tesseract), cryptographic routines (libsodium), and local plagiarism fingerprinting.

### Platform Implementations
- **Mobile (Android/iOS)**: Jetpack Compose Multiplatform UI, leveraging SwiftUI wrappers on iOS. Integrates device camera scanning, Apple Pencil/S Pen support, and push notifications (Firebase/APNs).
- **Desktop (Windows/macOS/Linux)**: Tauri-based shell embedding the web client with access to native file system, printers, and OS-level sharing.
- **Web**: Next.js (React 18) front-end using GraphQL queries (Apollo) and WebRTC for real-time collaboration.

### Offline-First Strategy
- Local SQLite databases (SQLDelight) synchronized via the Delta Sync service.
- Encrypted note vault stored using libsodium sealed boxes, with biometric unlock on supported devices.
- Conflict resolution using CRDTs (Yjs) to support collaborative editing even when offline.

## Server Architecture

### API Gateway
- Envoy Proxy terminates TLS, validates OAuth tokens, performs rate limiting, and routes requests to backend services.
- Supports HTTP/2 for gRPC services and HTTP/1.1 for REST/GraphQL endpoints.

### Microservices
1. **Ingestion Service**
   - Uploads handwritten scans or digital documents.
   - Performs preprocessing (deskewing, denoising) using OpenCV.
   - Publishes `note.ingested` events to Kafka.

2. **OCR Service**
   - Listens to ingestion events, applies language-specific OCR pipelines (Tesseract, Google Vision fallback).
   - Outputs structured text with bounding boxes, handwriting confidence scores, and stores intermediate artifacts in S3-compatible object storage.

3. **AI Rewriting Service**
   - Applies transformer models (OpenAI GPT fine-tune or in-house LLaMA derivative) to paraphrase, summarize, and create flashcards.
   - Maintains style presets (academic, concise, exam-ready) and ensures meaning preservation through semantic similarity scoring (Sentence-BERT).

4. **Plagiarism Detection Service**
   - Uses combined local fingerprinting (w-shingling) and external API (e.g., Turnitin integration) to compute similarity percentages.
   - Exposes results via gRPC streaming for live updates and writes verdicts to PostgreSQL.

5. **Collaboration Service**
   - Hosts WebRTC signaling, manages CRDT state distribution, and enforces access control lists for group workspaces.
   - Provides version history, inline commenting, and teacher portal features (assignment distribution, grading annotations).

6. **Export Service**
   - Generates PDF, DOCX, PPT exports via serverless functions (AWS Lambda) triggered by message bus events.
   - Applies watermarking and embeds usage metadata for traceability.

7. **Notification Service**
   - Delivers multi-channel notifications (email via SES, push via FCM/APNs, in-app via WebSockets) and reminders for review schedules.

### Data Layer
- **PostgreSQL**: Stores normalized metadata (users, notes, assignments, audit trails).
- **Redis**: Session cache, rate limits, presence information for real-time collaboration.
- **S3-Compatible Object Storage**: Raw uploads, processed documents, exported files with lifecycle policies.
- **Neo4j Graph Database**: Knowledge graph linking concepts, flashcards, and teacher curricula for recommendation features.
- **ElasticSearch**: Full-text search across notes and shared libraries.

### Security & Compliance
- Zero-trust networking within Kubernetes (mTLS between services using SPIFFE/SPIRE).
- Field-level encryption for sensitive data, key management through HashiCorp Vault.
- Audit logging forwarded to a SIEM (Splunk) with anomaly detection.
- SOC 2 Type II readiness with regular pen tests and vulnerability scans.

## AI & ML Pipeline
- **Model Training**: Hosted on dedicated GPU nodes; uses Kubeflow pipelines for data preprocessing, experiment tracking (MLflow), and model deployment (Seldon Core).
- **Content Safety**: Toxicity and PII detection filters applied to generated content before delivery.
- **Evaluation**: Human-in-the-loop review dashboard for teacher feedback and continuous improvement.

## Deployment & DevOps
- Infrastructure as Code with Terraform targeting AWS (primary) and Azure (secondary) with active-active failover.
- Kubernetes (EKS/AKS) for microservices, with ArgoCD for GitOps deployment.
- CI/CD using GitHub Actions: linting, unit/integration tests, SAST/DAST, automated mobile builds via Fastlane, desktop builds via Tauri bundler.
- Observability stack: Prometheus, Grafana, OpenTelemetry tracing, Loki logs.

## Scalability Considerations
- Horizontal auto-scaling for stateless services via HPA tied to CPU/GPU metrics.
- Asynchronous processing via Kafka topics and worker pools to handle bursty workloads (exam periods).
- Content Delivery Network (CloudFront) for static assets and exported files.
- Edge caching for AI model responses to reduce latency while preserving personalization constraints.

## Future Enhancements
- Adaptive learning recommendations leveraging reinforcement learning from student interactions.
- Federated learning for on-device personalization without sharing raw data.
- Integration with LMS systems (Canvas, Moodle, Google Classroom) via LTI 1.3.
- Blockchain-backed credentialing for teacher-issued certifications.

