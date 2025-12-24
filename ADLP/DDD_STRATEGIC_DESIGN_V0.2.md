# Domain-Driven Design: Strategic Design Document
## AI Data Labeling Platform (ADLP)

**Document Version:** 0.2  
**Date:** 2025-12-20  
**Status:** Enhanced Draft for Review  
**Compliance:** ISO/IEC/IEEE 42010:2011 (Architecture Description)

---

## Document Control

| Role | Name | Signature | Date |
|------|------|-----------|------|
| **Author** | Architecture Team | | 2025-12-20 |
| **Reviewer** | Tech Lead | | |
| **Approver** | CTO | | |

**Change History:**

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | 2025-12-20 | Architecture Team | Initial strategic design |
| 0.2 | 2025-12-20 | Architecture Team | Enhanced with Design Thinking, Event Storming, SLA/NFR (ADR-007), Enterprise Governance |

---

## Table of Contents


1. [Executive Summary](#1-executive-summary)
   1.1 [Purpose](#11-purpose)
   1.2 [Scope](#12-scope)
   1.3 [Stakeholders](#13-stakeholders)
   1.4 [Key Decisions](#14-key-decisions)
   1.5 [Design Thinking & Rationale](#15-design-thinking--rationale)
2. [Domain Analysis](#2-domain-analysis)
   2.1 [Problem Domain](#21-problem-domain)
   2.2 [Subdomains Classification](#22-subdomains-classification)
   2.3 [Domain Vision Statement](#23-domain-vision-statement)
   2.4 [Business Capabilities Map](#24-business-capabilities-map)
   2.5 [Event Storming](#25-event-storming-end-to-end-workflow-visualization)
3. [Bounded Contexts](#3-bounded-contexts)
4. [Context Mapping](#4-context-mapping)
5. [Ubiquitous Language](#5-ubiquitous-language)
6. [Strategic Patterns](#6-strategic-patterns)
7. [Domain Model](#7-domain-model)
8. [Architecture Decision Records](#8-architecture-decision-records)
   - ADR-001 through ADR-007
9. [Implementation Roadmap](#9-implementation-roadmap)
10. [Compliance & Standards](#10-compliance--standards)
    10.1 [ISO/IEC/IEEE 42010 Compliance](#101-isoiecieee-42010-compliance)
    10.2 [DDD Best Practices](#102-ddd-best-practices-compliance)
    10.3 [Microservices Best Practices](#103-microservices-best-practices)
    10.4 [Security & Privacy Standards](#104-security--privacy-standards)
    10.5 [Enterprise Governance](#105-enterprise-governance--quality-assurance)

---

## 1. Executive Summary

### 1.1 Purpose

This document defines the **strategic design** for the AI Data Labeling Platform (ADLP) using Domain-Driven Design (DDD) principles. It establishes:

- **Bounded Contexts** and their boundaries
- **Context Maps** showing relationships between contexts
- **Ubiquitous Language** for each domain
- **Strategic patterns** for integration
- **Domain models** at the conceptual level

### 1.2 Scope

**In Scope:**
- Strategic domain modeling (Bounded Contexts, Context Maps)
- High-level domain models (Aggregates, Entities, Value Objects)
- Integration patterns between contexts
- Ubiquitous language definitions

**Out of Scope:**
- Tactical implementation details (repositories, services implementation)
- Infrastructure concerns (databases, messaging specifics)
- UI/UX design
- Detailed API specifications

### 1.3 Stakeholders

| Stakeholder | Interest | Concerns |
|-------------|----------|----------|
| **Domain Experts** | Business logic correctness | Model reflects real-world processes |
| **Architects** | System structure | Maintainability, scalability |
| **Developers** | Implementation guidance | Clear boundaries, well-defined interfaces |
| **Product Owners** | Feature delivery | Business value, time-to-market |
| **Operations** | System reliability | Deployability, observability |

### 1.4 Key Decisions

> [!IMPORTANT]
> **Strategic Decisions:**
> 1. **9 Bounded Contexts** identified based on business capabilities
> 2. **Microservices Architecture** with one service per bounded context
> 3. **Event-Driven Integration** using Domain Events
> 4. **Shared Kernel** for common value objects (minimal)
> 5. **Anti-Corruption Layers** for external system integration

---


### 1.5 Design Thinking & Rationale

This section documents the **thought process** behind our strategic design decisions, providing transparency into how we arrived at the current architecture.

#### 1.5.1 Discovery Phase: Understanding the Problem Space

**Initial Questions:**
1. What makes data labeling complex at scale?
2. Where is the competitive advantage?
3. What are the natural boundaries in the domain?

**Key Insights:**
- **Quality is the differentiator**, not just throughput
- **Skill-based routing** is critical for efficiency
- **Marketplace dynamics** require financial integrity
- **AI prelabeling** reduces cost but needs orchestration

**Decision:** Focus Core Domains on Quality, Assignment, and Prelabeling

---

#### 1.5.2 Context Identification: The Iterative Process

**Iteration 1: Initial Decomposition (Too Coarse)**

```
Initial attempt: 3 large contexts
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Data      │  │  Labeling   │  │   Platform  │
│  Pipeline   │  │  Workflow   │  │   Services  │
└─────────────┘  └─────────────┘  └─────────────┘
```

**Problems Identified:**
- "Platform Services" too vague - mixed concerns
- "Labeling Workflow" conflates assignment and execution
- No clear ownership boundaries

**Iteration 2: Refined Decomposition (Too Fine)**

```
Attempt 2: 12 micro-contexts
- Auth, User, Profile, Skills, Rating, Wallet...
```

**Problems Identified:**
- Too granular - operational overhead
- Skills/Rating/Profile are cohesive - should be together
- Violates "team per context" principle

**Iteration 3: Final Design (9 Contexts)**

**Rationale for Each Context:**

| Context | Why Separate? | Why Not Merged? |
|---------|---------------|-----------------|
| **Identity & Access** | Generic domain, security team ownership | Auth is stable, rarely changes with business logic |
| **User Profile** | Supporting domain, user management team | Skills/Rating are cohesive, change together |
| **Wallet & Payment** | Financial integrity, compliance requirements | Must be isolated for audit, different team (Finance) |
| **Data Ingestion** | Data engineering concern, different cadence | Could merge with Prelabeling but different teams |
| **Prelabeling** | Core domain, ML engineering expertise | High complexity, frequent model updates |
| **Task Assignment** | Core domain, complex routing logic | Could merge with Labeling but different change drivers |
| **Labeling** | User-facing, frontend team ownership | UI/UX changes frequently, separate from backend logic |
| **Quality Assurance** | Core domain, proprietary algorithms | Most complex, needs isolation for IP protection |
| **Dataset Export** | Supporting domain, data engineering | Simple, low change frequency, could merge but clean separation |

---

#### 1.5.3 Integration Pattern Selection: The Decision Tree

**For each context relationship, we asked:**

```
┌─────────────────────────────────────┐
│ Are they owned by the same team?    │
└────────┬────────────────────────────┘
         │
    ┌────┴────┐
    │   YES   │──► Consider Shared Kernel or Partnership
    └─────────┘
         │
    ┌────┴────┐
    │   NO    │──► Continue...
    └────┬────┘
         │
┌────────┴────────────────────────────┐
│ Is upstream stable and trusted?     │
└────────┬────────────────────────────┘
         │
    ┌────┴────┐
    │   YES   │──► Conformist
    └─────────┘
         │
    ┌────┴────┐
    │   NO    │──► Continue...
    └────┬────┘
         │
┌────────┴────────────────────────────┐
│ Does downstream drive requirements? │
└────────┬────────────────────────────┘
         │
    ┌────┴────┐
    │   YES   │──► Customer/Supplier
    └─────────┘
         │
    ┌────┴────┐
    │   NO    │──► Continue...
    └────┬────┘
         │
┌────────┴────────────────────────────┐
│ Is it an external system?           │
└────────┬────────────────────────────┘
         │
    ┌────┴────┐
    │   YES   │──► Anti-Corruption Layer
    └─────────┘
         │
    ┌────┴────┐
    │   NO    │──► Open Host Service
    └─────────┘
```

**Examples of Application:**

**Identity → User Profile:**
- Same team? NO
- Upstream stable? YES → **Conformist**
- Rationale: Identity is generic, well-defined, rarely changes

**User Profile → Task Assignment:**
- Same team? NO
- Upstream stable? YES
- Downstream drives requirements? YES → **Customer/Supplier**
- Rationale: Assignment needs specific search API, drives design

**Data Ingestion ↔ Prelabeling:**
- Same team? YES → **Partnership**
- Rationale: Data Engineering owns both, tight workflow coupling
- Note: Could become Shared Kernel if more integration needed

---

#### 1.5.4 Aggregate Design Philosophy

**Principle:** Aggregates are **consistency boundaries**, not just object graphs.

**Decision Criteria:**
1. **Transactional Consistency:** What must change together atomically?
2. **Invariant Enforcement:** What rules must always hold?
3. **Performance:** Smaller aggregates = better concurrency

**Example: Why Batch is an Aggregate Root**

```
❌ WRONG: Large Aggregate
Batch
├── Assignment
├── Tasks (collection)
├── Segments (collection)  ← TOO LARGE
├── Transcripts (collection)  ← WRONG BOUNDARY
└── QualityScores

Problem: Transcripts change frequently (autosave)
         Locks entire Batch, poor concurrency
```

```
✅ CORRECT: Focused Aggregate
Batch (Assignment Context)
├── Assignment
└── Tasks (collection)

LabelingSession (Labeling Context)
├── Transcript (with versions)
└── Edits (collection)

QualityEvaluation (Quality Context)
├── QualityScore
└── Reviews (collection)

Rationale: Each aggregate has clear consistency boundary
           Changes are localized, better performance
```

---

#### 1.5.5 Event-Driven vs Synchronous: The Trade-offs

**Decision Matrix:**

| Scenario | Chosen Approach | Rationale |
|----------|----------------|-----------|
| **Prelabel → Assignment** | Event-Driven | Async workflow, no immediate response needed |
| **Assignment → User Profile (search)** | Synchronous REST | Need immediate results for routing decision |
| **Quality → Wallet (credit)** | Event-Driven | Financial transaction, needs audit trail |
| **Quality → User Profile (rating)** | Synchronous REST | Need confirmation before proceeding |

**Why Not All Events?**
- Synchronous queries are simpler for read-heavy operations
- Some operations need immediate feedback (e.g., search)
- Events add complexity (eventual consistency, ordering)

**Why Not All Synchronous?**
- Tight coupling
- Cascading failures
- No audit trail
- Difficult to scale

**Hybrid Approach:**
- **Commands:** Synchronous when immediate response needed
- **Events:** Asynchronous for workflows and notifications
- **Queries:** Synchronous for reads, CQRS for complex queries

---

## 2. Domain Analysis

### 2.1 Problem Domain

**Core Problem:**
Enable efficient, high-quality data labeling at scale through a marketplace model that connects data consumers with skilled labelers, leveraging AI for prelabeling and quality assurance.

**Domain Complexity:**
- **High:** Quality assurance, bias detection, skill-based routing
- **Medium:** Task assignment, prelabeling orchestration
- **Low:** User authentication, file storage

### 2.2 Subdomains Classification

Following DDD principles, we classify subdomains into **Core**, **Supporting**, and **Generic**:

#### 2.2.1 Core Domains (Competitive Advantage)

| Subdomain | Why Core | Business Impact |
|-----------|----------|-----------------|
| **Quality Assurance** | Proprietary algorithms for WER, overlap sampling, bias detection | Differentiates platform quality |
| **Task Assignment & Routing** | Intelligent routing based on skills, rating, confidence | Optimizes labeler productivity |
| **Prelabeling Orchestration** | Model selection, confidence scoring, segmentation | Reduces labeling cost by 70% |

#### 2.2.2 Supporting Domains (Business-Specific)

| Subdomain | Why Supporting | Business Impact |
|-----------|----------------|-----------------|
| **User Profile Management** | Labeler skills, ratings, preferences | Enables core routing logic |
| **Wallet & Payout** | Earnings, withdrawals, invoicing | Enables marketplace model |
| **Data Ingestion** | Audio normalization, validation | Ensures data quality |

#### 2.2.3 Generic Domains (Commodity)

| Subdomain | Why Generic | Solution |
|-----------|-------------|----------|
| **Authentication & Authorization** | Standard JWT + RBAC | Use proven libraries/services |
| **File Storage** | Standard object storage | AWS S3 |
| **Notification** | Email, push notifications | AWS SES, SNS |

### 2.3 Domain Vision Statement

> **Vision:**
> 
> ADLP is the **most trusted and efficient** AI data labeling platform, delivering **enterprise-grade quality** through:
> - **AI-powered prelabeling** that reduces manual effort by 70%
> - **Intelligent quality assurance** ensuring >95% accuracy
> - **Fair marketplace** rewarding labelers based on skill and quality
> - **Bias-free datasets** meeting ethical AI standards

### 2.4 Business Capabilities Map

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI Data Labeling Platform                     │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ Data Supply  │      │   Labeling   │      │   Quality    │
│              │      │   Workflow   │      │  Assurance   │
└──────────────┘      └──────────────┘      └──────────────┘
        │                     │                     │
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ • Ingestion  │      │ • Assignment │      │ • Quality    │
│ • Prelabeling│      │ • Labeling   │      │   Metrics    │
│ • Storage    │      │ • Submission │      │ • Review     │
└──────────────┘      └──────────────┘      │ • Bias Check │
                                            └──────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                              ▼
                      ┌──────────────┐
                      │  Marketplace │
                      │              │
                      └──────────────┘
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
        ┌──────────────┐            ┌──────────────┐
        │ User & Skills│            │   Wallet &   │
        │  Management  │            │    Payout    │
        └──────────────┘            └──────────────┘
```

---


### 2.5 Event Storming: End-to-End Workflow Visualization

Event Storming is a collaborative workshop technique to discover domain events and workflows. This section presents the **timeline view** of our core business process.

#### 2.5.1 Complete Labeling Workflow

```
Timeline: Data Ingestion (Upload/Crawl) → Dataset Export
═══════════════════════════════════════════════════════════════════════

[ACTOR]         [COMMAND]              [EVENT]                [AGGREGATE]
───────────────────────────────────────────────────────────────────────

═══════════════════════════════════════════════════════════════════════
WORKFLOW 1: SCHEDULED CRAWL
═══════════════════════════════════════════════════════════════════════

Scheduler  ──► TriggerCrawl        ──► CrawlScheduled       ──► CrawlJob
                                        │
                                        ▼
Crawler    ──► FetchData           ──► DataFetched          ──► DataItem
                                        │
                                        ▼
System     ──► ValidateData        ──► DataValidated        ──► DataItem
                                        │
                                        ▼
System     ──► NormalizeData       ──► DataNormalized       ──► DataItem
                                        │
                                        ▼
System     ──► CreatePrelabelJob   ──► PrelabelJobCreated   ──► PrelabelJob
                                        │
                                        └──► (continues to Workflow 2)

═══════════════════════════════════════════════════════════════════════
WORKFLOW 2: MANUAL UPLOAD
═══════════════════════════════════════════════════════════════════════


[ACTOR]         [COMMAND]              [EVENT]                [AGGREGATE]
───────────────────────────────────────────────────────────────────────

Admin      ──► UploadData          ──► DataUploaded          ──► DataItem
                                        │
                                        ▼
System     ──► NormalizeData      ──► DataNormalized       ──► DataItem
                                        │
                                        ▼
System     ──► CreatePrelabelJob   ──► PrelabelJobCreated    ──► PrelabelJob
                                        │
                                        ▼
SageMaker  ──► RunInference        ──► PrelabelCompleted     ──► PrelabelJob
                                        │
                                        ▼
System     ──► CreateBatch         ──► BatchCreated          ──► Batch
                                        │
                                        ▼
System     ──► CalculatePriority   ──► PriorityCalculated    ──► Batch
                                        │
                                        ▼
System     ──► RouteBatch          ──► BatchRouted           ──► Batch
                                        │
                                        ▼
Labeler    ──► AcceptBatch         ──► BatchAssigned         ──► Batch
                                        │
                                        ▼
Labeler    ──► EditTranscript      ──► TranscriptEdited      ──► LabelingSession
           │                            │
           │   (autosave every 5s)      │
           └────────────────────────────┘
                                        │
                                        ▼
Labeler    ──► SubmitBatch         ──► BatchSubmitted        ──► Batch
                                        │
                                        ▼
System     ──► EvaluateQuality     ──► QualityEvaluated      ──► QualityEvaluation
                                        │
                                    ┌───┴───┐
                                    │       │
                            Quality │       │ Quality
                              Good  │       │  Poor
                                    │       │
                                    ▼       ▼
                              BatchAccepted  ReviewRequired
                                    │             │
                                    │             ▼
                                    │       Reviewer ──► ReviewBatch
                                    │             │
                                    │             ▼
                                    │       ReviewCompleted
                                    │             │
                                    └─────────┬───┘
                                              ▼
System     ──► UpdateRating        ──► RatingUpdated         ──► Labeler
                                        │
                                        ▼
System     ──► CreditEarnings      ──► EarningCredited       ──► Wallet
                                        │
                                        ▼
Consumer   ──► RequestExport       ──► ExportRequested       ──► Dataset
                                        │
                                        ▼
System     ──► BuildDataset        ──► DatasetBuilt          ──► Dataset
                                        │
                                        ▼
System     ──► GenerateSignedURL   ──► ExportCompleted       ──► Dataset

═══════════════════════════════════════════════════════════════════════
```

#### 2.5.2 Event Correlation & Causation

**Event Metadata Structure:**

```json
{
  "event_id": "550e8400-e29b-41d4-a716-446655440000",
  "event_type": "BatchSubmitted",
  "timestamp": "2025-12-20T12:00:00Z",
  "version": "1.0",
  "correlation_id": "batch-workflow-12345",  ← Links all events in workflow
  "causation_id": "event-id-of-BatchAssigned",  ← Direct cause
  "aggregate_id": "batch-uuid",
  "aggregate_type": "Batch",
  "payload": { ... }
}
```

**Correlation Example:**

```
correlation_id: "data-item-abc123"

DataUploaded (event_id: e1, causation_id: null)
    ↓
DataNormalized (event_id: e2, causation_id: e1)
    ↓
PrelabelJobCreated (event_id: e3, causation_id: e2)
    ↓
PrelabelCompleted (event_id: e4, causation_id: e3)
    ↓
BatchCreated (event_id: e5, causation_id: e4)
    ↓
... (all events share correlation_id: "data-item-abc123")
```

**Benefits:**
- **Distributed Tracing:** Follow workflow across services
- **Debugging:** Identify where workflow failed
- **Audit:** Complete history of item lifecycle
- **Analytics:** Measure end-to-end latency

---

#### 2.5.3 Hotspots & Bottlenecks

**Identified During Event Storming:**

| Hotspot | Issue | Mitigation |
|---------|-------|------------|
| **PrelabelCompleted → BatchCreated** | High volume (360k segments/week) | Batch processing, queue buffering |
| **BatchSubmitted → QualityEvaluated** | Complex calculation (WER, overlap) | Async workers, horizontal scaling |
| **ReviewRequired → ReviewCompleted** | Human bottleneck | Priority queue, SLA monitoring |
| **ExportRequested → DatasetBuilt** | Large dataset compilation | Async job, progress tracking |

---

## 3. Bounded Contexts

### 3.1 Context Identification Criteria

We identify Bounded Contexts based on:
1. **Business capability** boundaries
2. **Team ownership** (Conway's Law)
3. **Data ownership** and lifecycle
4. **Change frequency** and reasons
5. **Ubiquitous language** variations

### 3.2 Bounded Context Catalog

#### BC1: Identity & Access Context

**Business Capability:** User authentication and authorization

**Ubiquitous Language:**
- **User:** A person or system with credentials
- **Credential:** Email + password or API key
- **Role:** Labeler, Reviewer, Admin, Consumer
- **Permission:** Fine-grained access right
- **Session:** Authenticated user state
- **Token:** JWT representing authenticated session

**Core Concepts:**
- User (Aggregate Root)
- Credential (Entity)
- Role (Value Object)
- Permission (Value Object)
- Session (Entity)

**Responsibilities:**
- User registration and email verification
- Authentication (login, logout, token refresh)
- Authorization (RBAC enforcement)
- Password management (reset, change)
- Session management

**Boundaries:**
- **In:** Authentication, authorization, user credentials
- **Out:** User profiles, skills, ratings (belongs to User Profile Context)

**Team:** Security Team

---

#### BC2: User Profile Context

**Business Capability:** Labeler and consumer profile management

**Ubiquitous Language:**
- **Labeler:** A user who performs labeling tasks
- **Profile:** Personal information and preferences
- **Skill:** A competency in a specific area (language, data type, domain)
- **Proficiency:** Level of skill mastery (beginner, intermediate, expert)
- **Rating:** Elo-based performance score
- **Certification:** Verified skill credential
- **Preference:** User settings for notifications and UI

**Core Concepts:**
- Labeler (Aggregate Root)
- Profile (Entity)
- Skill (Value Object)
- Rating (Entity with history)
- Certification (Entity)
- Preference (Value Object)

**Responsibilities:**
- Profile CRUD operations
- Skill management and verification
- Rating calculation and history tracking
- Performance statistics aggregation
- Labeler search and discovery

**Boundaries:**
- **In:** Profiles, skills, ratings, preferences
- **Out:** Authentication (belongs to Identity Context), wallet balance (belongs to Wallet Context)

**Invariants:**
- Rating must be between 0 and 3000
- Skills must be verified before used for routing
- Profile must be complete before labeling

**Team:** User Management Team

---

#### BC3: Wallet & Payment Context

**Business Capability:** Financial transactions and payouts

**Ubiquitous Language:**
- **Wallet:** A labeler's account for earnings
- **Balance:** Current available funds
- **Pending Balance:** Earnings not yet cleared
- **Transaction:** A financial event (earning, withdrawal, bonus, penalty)
- **Earning:** Payment for completed work
- **Withdrawal:** Request to transfer funds out
- **Payout:** Completed withdrawal to external account
- **Invoice:** Receipt for earnings

**Core Concepts:**
- Wallet (Aggregate Root)
- Transaction (Entity)
- WithdrawalRequest (Entity)
- Invoice (Entity)
- Money (Value Object)

**Responsibilities:**
- Wallet balance management
- Transaction recording and history
- Withdrawal request processing
- Payment provider integration
- Invoice generation

**Boundaries:**
- **In:** Wallets, transactions, withdrawals, invoices
- **Out:** User identity (belongs to Identity Context), batch quality (belongs to Quality Context)

**Invariants:**
- Balance cannot be negative
- Withdrawal amount must not exceed available balance
- Transactions must be immutable once completed

**Team:** Finance Team

---

#### BC4: Data Ingestion Context

**Business Capability:** Data acquisition and preprocessing

**Ubiquitous Language:**
- **DataItem:** A raw data file (audio, image, text)
- **AudioFile:** Audio data subtype
- **ImageFile:** Image data subtype
- **TextFile:** Text data subtype
- **Normalization:** Converting to standard format
- **Validation:** Checking format, size, integrity
- **Metadata:** Descriptive information about data
- **Hash:** Unique identifier for deduplication
- **Source:** Origin of data (upload, crawl, API)

**Core Concepts:**
- DataItem (Aggregate Root)
- AudioFile, ImageFile, TextFile (Entities, subtypes of DataItem)
- Metadata (Value Object)
- ValidationResult (Value Object)
- NormalizationSpec (Value Object)

**Responsibilities:**
- Data upload and crawling
- Format validation
- Data normalization (format-specific: audio 16kHz mono, image resize, etc.)
- Deduplication via hashing
- Metadata extraction and storage
- Triggering prelabeling workflow

**Boundaries:**
- **In:** Raw data ingestion, validation, normalization
- **Out:** Prelabeling (belongs to Prelabeling Context), storage location (infrastructure)

**Invariants:**
- Data must be normalized before prelabeling
- Duplicate data items must be rejected
- Metadata must be persisted atomically with file

**Team:** Data Engineering Team

---

#### BC5: Prelabeling Context

**Business Capability:** AI-powered automatic labeling

**Ubiquitous Language:**
- **PrelabelJob:** A request to prelabel data
- **Model:** An AI model for prelabeling (Whisper, DETR, etc.)
- **Inference:** Running model on data
- **Segment:** A chunk of data (10s audio, image region)
- **Prediction:** Model output with confidence
- **Confidence:** Model's certainty in prediction (0-1)
- **ModelRouter:** Logic to select appropriate model

**Core Concepts:**
- PrelabelJob (Aggregate Root)
- Segment (Entity)
- Prediction (Value Object)
- Model (Entity)
- InferenceResult (Value Object)

**Responsibilities:**
- Model selection and routing
- Inference execution (via SageMaker)
- Segmentation (audio → 10s chunks)
- Confidence scoring
- Prelabel result persistence
- Triggering assignment workflow

**Boundaries:**
- **In:** Prelabeling orchestration, model management, inference
- **Out:** Data storage (belongs to Ingestion Context), task assignment (belongs to Assignment Context)

**Invariants:**
- Segments must not overlap
- Confidence must be between 0 and 1
- Model version must be tracked for reproducibility

**Team:** ML Engineering Team

---

#### BC6: Task Assignment Context

**Business Capability:** Intelligent task routing and assignment

**Ubiquitous Language:**
- **Batch:** A collection of segments for labeling
- **Task:** A single labeling assignment
- **Assignment:** Linking a batch to a labeler
- **Queue:** Priority-ordered list of available batches
- **Priority:** Urgency score based on tier, deadline, difficulty
- **Lock:** Temporary reservation of batch
- **Routing:** Algorithm to match batches with labelers
- **Deadline:** Time limit for batch completion

**Core Concepts:**
- Batch (Aggregate Root)
- Assignment (Entity)
- Task (Entity)
- Queue (Entity)
- RoutingRule (Value Object)
- Priority (Value Object)

**Responsibilities:**
- Batch creation from prelabeled segments
- Priority calculation
- Skill-based routing
- Batch locking and unlocking
- TTL-based auto-reassignment
- Notification of available tasks

**Boundaries:**
- **In:** Batch lifecycle, assignment, routing, queue management
- **Out:** Prelabel results (belongs to Prelabeling Context), labeler skills (belongs to User Profile Context)

**Invariants:**
- A batch can only be assigned to one labeler at a time
- Lock must expire after TTL
- Priority must be recalculated on status change

**Team:** Workflow Team

---

#### BC7: Labeling Context

**Business Capability:** Human labeling interface and data capture

**Ubiquitous Language:**
- **LabelingSession:** A labeler working on a batch
- **Transcript:** Text transcription of audio
- **Edit:** Change made to prelabel
- **Flag:** Issue reported by labeler
- **Submission:** Completed batch sent for review
- **Autosave:** Periodic save of work in progress
- **Version:** Iteration of transcript

**Core Concepts:**
- LabelingSession (Aggregate Root)
- Transcript (Entity with versioning)
- Edit (Value Object)
- Flag (Entity)
- Submission (Event)

**Responsibilities:**
- Serving segments and prelabels to UI
- Capturing transcript edits
- Autosave functionality
- Issue flagging
- Batch submission
- Edit history tracking

**Boundaries:**
- **In:** Labeling workflow, transcript editing, submission
- **Out:** Batch assignment (belongs to Assignment Context), quality evaluation (belongs to Quality Context)

**Invariants:**
- Transcript versions must be immutable
- Submission must include all segments
- Edits must be attributed to labeler

**Team:** Frontend Team + Backend API Team

---

#### BC8: Quality Assurance Context

**Business Capability:** Quality measurement and review

**Ubiquitous Language:**
- **QualityScore:** Aggregate measure of batch quality
- **WER:** Word Error Rate (for audio)
- **Agreement:** Overlap similarity between labelers
- **Overlap:** Segments labeled by multiple labelers
- **Review:** Human evaluation of flagged batch
- **Escalation:** Promotion to higher review level
- **Anomaly:** Statistical outlier in quality metrics
- **Bias:** Demographic imbalance in dataset
- **Fairness:** Equalized odds across groups

**Core Concepts:**
- QualityEvaluation (Aggregate Root)
- QualityScore (Entity)
- Review (Entity)
- Anomaly (Value Object)
- BiasReport (Entity)
- Escalation (Entity)

**Responsibilities:**
- WER calculation (before/after labeling)
- Overlap sampling and agreement scoring
- Labeler rating updates
- Automated anomaly detection
- Bias detection (Fairlearn)
- Review workflow and escalation
- Quality metrics aggregation

**Boundaries:**
- **In:** Quality metrics, reviews, bias detection, escalation
- **Out:** Transcript data (belongs to Labeling Context), rating updates (belongs to User Profile Context)

**Invariants:**
- Quality score must be between 0 and 1
- Reviews must have audit trail
- Bias metrics must be computed for all datasets

**Team:** Quality Team

---

#### BC9: Dataset Export Context

**Business Capability:** Dataset packaging and distribution

**Ubiquitous Language:**
- **Dataset:** A collection of labeled data
- **Export:** Process of packaging dataset
- **Format:** Output structure (JSON, CSV, TFRecord)
- **Version:** Snapshot of dataset at a point in time
- **SignedURL:** Time-limited download link
- **Watermark:** Ownership marker for copyright
- **License:** Usage terms for dataset

**Core Concepts:**
- Dataset (Aggregate Root)
- Export (Entity)
- Version (Value Object)
- License (Value Object)
- DownloadToken (Value Object)

**Responsibilities:**
- Dataset compilation from accepted transcripts
- Format conversion (JSON, CSV)
- Versioning and changelog
- Signed URL generation
- Watermarking (optional)
- Download tracking

**Boundaries:**
- **In:** Dataset creation, export, versioning, distribution
- **Out:** Transcript data (belongs to Labeling Context), quality scores (belongs to Quality Context)

**Invariants:**
- Exports must be immutable once created
- Signed URLs must expire
- Only accepted transcripts can be exported

**Team:** Data Engineering Team

---

### 3.3 Context Comparison Matrix

| Context | Core/Supporting/Generic | Complexity | Change Frequency | Team Size |
|---------|-------------------------|------------|------------------|-----------|
| Identity & Access | Generic | Low | Low | 1-2 |
| User Profile | Supporting | Medium | Medium | 2-3 |
| Wallet & Payment | Supporting | Medium | Low | 2-3 |
| Data Ingestion | Supporting | Low | Low | 2-3 |
| Prelabeling | **Core** | High | High | 3-4 |
| Task Assignment | **Core** | High | High | 3-4 |
| Labeling | Supporting | Medium | Medium | 3-4 |
| Quality Assurance | **Core** | High | High | 3-4 |
| Dataset Export | Supporting | Low | Low | 1-2 |

---

## 4. Context Mapping

### 4.1 Context Map Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Context Map Legend                           │
├─────────────────────────────────────────────────────────────────────┤
│  U/D  = Upstream/Downstream                                         │
│  P/C  = Published Language / Conformist                             │
│  ACL  = Anti-Corruption Layer                                       │
│  SK   = Shared Kernel                                               │
│  OHS  = Open Host Service                                           │
│  ──►  = Dependency direction                                        │
└─────────────────────────────────────────────────────────────────────┘

                    ┌──────────────────┐
                    │   Identity &     │
                    │  Access Context  │ [OHS]
                    └────────┬─────────┘
                             │ U
                    ┌────────┴─────────┐
                    │                  │
                    │ D                │ D
            ┌───────▼──────┐   ┌──────▼────────┐
            │ User Profile │   │    Wallet &   │
            │   Context    │   │Payment Context│
            └───────┬──────┘   └───────┬───────┘
                    │ U                │ U
                    │                  │
            ┌───────┴──────────────────┴───────┐
            │                                  │
            │ D                                │ D
    ┌───────▼──────┐                   ┌──────▼────────┐
    │     Task     │◄──────────────────│   Quality     │
    │  Assignment  │        U/D        │  Assurance    │
    │   Context    │                   │   Context     │
    └───────┬──────┘                   └───────▲───────┘
            │ U                                │ U
            │                                  │
            │ D                        ┌───────┴───────┐
    ┌───────▼──────┐                   │               │
    │   Labeling   │───────────────────┘               │
    │   Context    │        U/D                        │
    └───────▲──────┘                                   │
            │ U                                        │
            │                                          │
    ┌───────┴──────┐                           ┌──────┴────────┐
    │ Prelabeling  │                           │Dataset Export │
    │   Context    │                           │   Context     │
    └───────▲──────┘                           └───────────────┘
            │ U
            │
    ┌───────┴──────┐
    │     Data     │
    │  Ingestion   │
    │   Context    │
    └──────────────┘
```

### 4.2 Context Relationships

#### 4.2.1 Identity & Access ──► User Profile (OHS/Conformist)

**Pattern:** Open Host Service / Conformist

**Relationship:**
- Identity & Access is **Upstream** (U)
- User Profile is **Downstream** (D)
- User Profile conforms to Identity's User model

**Integration:**
- **Published Language:** User ID (UUID), Email, Role
- **Protocol:** REST API + Domain Events
- **Events:** `UserRegistered`, `EmailVerified`, `UserSuspended`

**Rationale:**
- Identity owns the canonical User entity
- User Profile extends with domain-specific attributes
- No ACL needed as User Profile fully trusts Identity

**API Contract:**
```yaml
GET /users/{id}
Response:
  user_id: uuid
  email: string
  role: enum[labeler, reviewer, admin, consumer]
  status: enum[active, suspended, deleted]
  email_verified: boolean
```

---

#### 4.2.2 Identity & Access ──► Wallet & Payment (OHS/Conformist)

**Pattern:** Open Host Service / Conformist

**Relationship:**
- Identity & Access is **Upstream** (U)
- Wallet & Payment is **Downstream** (D)

**Integration:**
- **Published Language:** User ID
- **Protocol:** REST API + Domain Events
- **Events:** `UserRegistered`, `UserDeleted`

**Rationale:**
- Wallet needs User ID to create wallet
- Wallet doesn't need full user details

---

#### 4.2.3 User Profile ──► Task Assignment (Customer/Supplier)

**Pattern:** Customer/Supplier

**Relationship:**
- User Profile is **Upstream** (U)
- Task Assignment is **Downstream** (D) and **Customer**

**Integration:**
- **Published Language:** Labeler Skills, Rating
- **Protocol:** REST API (synchronous queries)
- **Events:** `SkillVerified`, `RatingUpdated`

**API Contract:**
```yaml
GET /users/search
Query:
  skills: [string]
  rating_min: integer
  rating_max: integer
  available: boolean
Response:
  labelers: [
    {
      user_id: uuid,
      rating: integer,
      skills: [skill],
      active_batches: integer
    }
  ]
```

**Rationale:**
- Assignment needs labeler skills for routing
- User Profile provides search API tailored to Assignment's needs
- Assignment drives the API design (customer)

---

#### 4.2.4 Quality Assurance ──► User Profile (Customer/Supplier)

**Pattern:** Customer/Supplier

**Relationship:**
- Quality Assurance is **Downstream** (D) and **Customer**
- User Profile is **Upstream** (U)

**Integration:**
- **Protocol:** REST API (rating updates)
- **Events:** `QualityEvaluated` → triggers rating update

**API Contract:**
```yaml
POST /users/{id}/rating
Request:
  new_rating: integer
  previous_rating: integer
  change_reason: string
  batch_id: uuid
  quality_score: decimal
```

**Rationale:**
- Quality owns the rating calculation logic
- User Profile stores the rating history
- Quality drives the update API

---

#### 4.2.5 Quality Assurance ──► Wallet & Payment (Customer/Supplier)

**Pattern:** Customer/Supplier

**Relationship:**
- Quality Assurance is **Downstream** (D)
- Wallet & Payment is **Upstream** (U)

**Integration:**
- **Protocol:** REST API (credit earnings)
- **Events:** `BatchAccepted` → triggers earning credit

**API Contract:**
```yaml
POST /wallets/{user_id}/credit
Request:
  amount: decimal
  reference_type: "batch"
  reference_id: uuid
  description: string
```

**Rationale:**
- Quality determines if batch is accepted and earnings
- Wallet records the transaction

---

#### 4.2.6 Data Ingestion ──► Prelabeling (Partnership)

**Pattern:** Partnership

**Relationship:**
- Bidirectional dependency
- Both contexts collaborate closely

**Integration:**
- **Protocol:** Kafka Events (async)
- **Events:** `DataItemIngested` → `PrelabelJobCreated` → `PrelabelCompleted`

**Rationale:**
- Tight coupling due to workflow dependency
- Both owned by Data Engineering team
- Could be merged into single context if team structure changes

---

#### 4.2.7 Prelabeling ──► Task Assignment (Customer/Supplier)

**Pattern:** Customer/Supplier

**Relationship:**
- Prelabeling is **Upstream** (U)
- Task Assignment is **Downstream** (D) and **Customer**

**Integration:**
- **Protocol:** Kafka Events
- **Events:** `PrelabelCompleted` → triggers batch creation

**Event Schema:**
```yaml
PrelabelCompleted:
  item_id: uuid
  segments: [
    {
      segment_id: uuid,
      start_time: decimal,
      end_time: decimal,
      prelabel_text: string,
      confidence: decimal
    }
  ]
  model_id: string
  avg_confidence: decimal
```

**Rationale:**
- Assignment needs prelabel results to create batches
- Assignment drives the event schema (customer)

---

#### 4.2.8 Task Assignment ──► Labeling (Customer/Supplier)

**Pattern:** Customer/Supplier

**Relationship:**
- Task Assignment is **Upstream** (U)
- Labeling is **Downstream** (D)

**Integration:**
- **Protocol:** REST API + Kafka Events
- **Events:** `BatchAssigned`, `BatchUnlocked`

**API Contract:**
```yaml
GET /batches/{id}/segments
Response:
  batch_id: uuid
  segments: [
    {
      segment_id: uuid,
      audio_url: string (signed),
      prelabel_text: string,
      confidence: decimal
    }
  ]
```

**Rationale:**
- Labeling consumes batches created by Assignment
- Assignment owns batch lifecycle

---

#### 4.2.9 Labeling ──► Quality Assurance (Partnership)

**Pattern:** Partnership

**Relationship:**
- Bidirectional dependency
- Labeling produces transcripts, Quality evaluates them

**Integration:**
- **Protocol:** Kafka Events + REST API
- **Events:** `BatchSubmitted` → `QualityEvaluated` → `ReviewRequired`

**Rationale:**
- Tight workflow coupling
- Quality may trigger re-labeling (feedback loop)

---

#### 4.2.10 Quality Assurance ──► Dataset Export (Customer/Supplier)

**Pattern:** Customer/Supplier

**Relationship:**
- Quality Assurance is **Upstream** (U)
- Dataset Export is **Downstream** (D)

**Integration:**
- **Protocol:** REST API (query accepted transcripts)
- **Events:** `BatchAccepted`

**Rationale:**
- Export only includes quality-approved data
- Quality determines what is exportable

---

### 4.3 Integration Patterns Summary

| Upstream Context | Downstream Context | Pattern | Protocol |
|------------------|-------------------|---------|----------|
| Identity & Access | User Profile | OHS/Conformist | REST + Events |
| Identity & Access | Wallet & Payment | OHS/Conformist | REST + Events |
| User Profile | Task Assignment | Customer/Supplier | REST + Events |
| User Profile | Quality Assurance | Customer/Supplier | REST |
| Quality Assurance | Wallet & Payment | Customer/Supplier | REST |
| Data Ingestion | Prelabeling | Partnership | Events |
| Prelabeling | Task Assignment | Customer/Supplier | Events |
| Task Assignment | Labeling | Customer/Supplier | REST + Events |
| Labeling | Quality Assurance | Partnership | Events + REST |
| Quality Assurance | Dataset Export | Customer/Supplier | REST |

---

## 5. Ubiquitous Language

### 5.1 Cross-Context Shared Terms

> [!NOTE]
> These terms have **consistent meaning** across all contexts:

| Term | Definition | Type |
|------|------------|------|
| **User** | A person or system with credentials | Entity |
| **User ID** | Unique identifier (UUID) for a user | Value Object |
| **Timestamp** | ISO 8601 datetime in UTC | Value Object |
| **Money** | Amount + Currency (e.g., 10.50 USD) | Value Object |
| **Status** | Lifecycle state (active, pending, completed, etc.) | Value Object |

### 5.2 Context-Specific Language

#### Identity & Access Context

```yaml
Entities:
  User: {id, email, password_hash, role, status}
  Session: {id, user_id, token, expires_at}

Value Objects:
  Credential: {email, password}
  Role: enum[labeler, reviewer, admin, consumer]
  Permission: {resource, action}
  Token: {jwt_string, expires_at}

Aggregates:
  User (root)
    - Credentials
    - Roles
    - Sessions

Domain Events:
  UserRegistered: {user_id, email, role, timestamp}
  EmailVerified: {user_id, timestamp}
  UserLoggedIn: {user_id, session_id, timestamp}
  UserSuspended: {user_id, reason, timestamp}
```

#### User Profile Context

```yaml
Entities:
  Labeler: {user_id, profile, skills, rating, preferences}
  Profile: {full_name, bio, avatar_url, country}
  Rating: {user_id, rating, history}

Value Objects:
  Skill: {type, value, proficiency_level}
  Proficiency: enum[beginner, intermediate, expert]
  RatingChange: {old_rating, new_rating, reason}

Aggregates:
  Labeler (root)
    - Profile
    - Skills (collection)
    - Rating (with history)
    - Preferences

Domain Events:
  ProfileUpdated: {user_id, fields_changed, timestamp}
  SkillAdded: {user_id, skill, timestamp}
  SkillVerified: {user_id, skill, verified_by, timestamp}
  RatingUpdated: {user_id, old_rating, new_rating, reason, timestamp}
```

#### Wallet & Payment Context

```yaml
Entities:
  Wallet: {user_id, balance, pending_balance, currency}
  Transaction: {id, user_id, type, amount, status}
  WithdrawalRequest: {id, user_id, amount, payment_method, status}

Value Objects:
  Money: {amount, currency}
  TransactionType: enum[earning, withdrawal, bonus, penalty]
  PaymentMethod: enum[bank_transfer, paypal, stripe]

Aggregates:
  Wallet (root)
    - Transactions (collection)
    - WithdrawalRequests (collection)

Domain Events:
  EarningCredited: {user_id, amount, batch_id, timestamp}
  WithdrawalRequested: {withdrawal_id, user_id, amount, timestamp}
  WithdrawalCompleted: {withdrawal_id, user_id, amount, timestamp}

Invariants:
  - balance >= 0
  - withdrawal_amount <= available_balance
  - transactions are immutable
```

#### Task Assignment Context

```yaml
Entities:
  Batch: {id, name, priority, status, deadline}
  Assignment: {id, batch_id, labeler_id, locked_until}
  Task: {id, batch_id, segment_id}

Value Objects:
  Priority: {score, tier, deadline_urgency, difficulty}
  QualityTier: enum[basic, standard, premium]
  Difficulty: enum[easy, medium, hard]

Aggregates:
  Batch (root)
    - Tasks (collection)
    - Assignment

Domain Events:
  BatchCreated: {batch_id, priority, segment_count, timestamp}
  BatchAssigned: {batch_id, labeler_id, locked_until, timestamp}
  BatchSubmitted: {batch_id, labeler_id, timestamp}
  BatchUnlocked: {batch_id, reason, timestamp}

Invariants:
  - batch can only be assigned to one labeler at a time
  - lock expires after TTL
  - priority recalculated on status change
```

#### Quality Assurance Context

```yaml
Entities:
  QualityEvaluation: {id, batch_id, scores, status}
  Review: {id, batch_id, reviewer_id, decision}
  BiasReport: {id, dataset_id, metrics}

Value Objects:
  QualityScore: {wer_before, wer_after, agreement, overall}
  ReviewDecision: enum[accept, reject, escalate]
  BiasMetric: {demographic_parity, equalized_odds}

Aggregates:
  QualityEvaluation (root)
    - QualityScores
    - Reviews (collection)

Domain Events:
  QualityEvaluated: {batch_id, quality_score, timestamp}
  BatchAccepted: {batch_id, quality_score, timestamp}
  BatchRejected: {batch_id, reason, timestamp}
  ReviewRequired: {batch_id, reason, priority, timestamp}
  BiasDetected: {dataset_id, bias_metrics, timestamp}

Invariants:
  - quality_score between 0 and 1
  - reviews must have audit trail
  - bias metrics computed for all datasets
```

### 5.3 Translation Map

When contexts communicate, terms may need translation:

| Source Context | Term | Target Context | Translated Term |
|----------------|------|----------------|-----------------|
| Identity & Access | User | User Profile | Labeler |
| User Profile | Rating | Task Assignment | LabelerRating |
| Prelabeling | Prediction | Labeling | PrelabelText |
| Labeling | Transcript | Quality Assurance | LabeledText |
| Quality Assurance | Accepted | Dataset Export | Exportable |

---

## 6. Strategic Patterns

### 6.1 Pattern Application

#### 6.1.1 Shared Kernel

**Applied Between:** Data Ingestion ↔ Prelabeling

**Shared Concepts:**
- `DataItem` (base entity)
- `Metadata` (value object)
- `Hash` (value object for deduplication)

**Rationale:**
- Both contexts work with same data model
- Owned by same team (Data Engineering)
- Changes require coordination

**Risks:**
- Tight coupling
- Coordinated releases required

**Mitigation:**
- Minimize shared kernel size
- Clear ownership and change process
- Consider splitting if teams diverge

---

#### 6.1.2 Customer/Supplier

**Applied Between:** User Profile (Supplier) → Task Assignment (Customer)

**Contract:**
- Customer (Assignment) defines API requirements
- Supplier (User Profile) implements to spec
- SLA: 99.9% availability, p95 latency < 100ms

**Governance:**
- Customer reviews API changes
- Supplier cannot break contract without customer approval
- Versioned API with deprecation policy

---

#### 6.1.3 Conformist

**Applied Between:** Identity & Access (Upstream) → User Profile (Downstream)

**Rationale:**
- User Profile fully trusts Identity's User model
- No translation needed
- Simplifies integration

**Trade-off:**
- User Profile is coupled to Identity's model
- Acceptable because Identity is stable and well-defined

---

#### 6.1.4 Anti-Corruption Layer (ACL)

**Applied Between:** ADLP ↔ External Payment Providers (Stripe, PayPal)

**Purpose:**
- Protect Wallet & Payment context from external model changes
- Translate external concepts to domain language

**Implementation:**
```python
# ACL for Stripe
class StripePaymentAdapter:
    def __init__(self, stripe_client):
        self.client = stripe_client
    
    def process_withdrawal(self, withdrawal_request: WithdrawalRequest) -> PayoutResult:
        # Translate domain model to Stripe API
        stripe_transfer = {
            'amount': int(withdrawal_request.amount.amount * 100),  # cents
            'currency': withdrawal_request.amount.currency.lower(),
            'destination': withdrawal_request.payment_details['stripe_account_id']
        }
        
        # Call external API
        result = self.client.transfers.create(**stripe_transfer)
        
        # Translate back to domain model
        return PayoutResult(
            success=result.status == 'paid',
            external_id=result.id,
            completed_at=datetime.fromtimestamp(result.created)
        )
```

---

#### 6.1.5 Open Host Service (OHS)

**Applied To:** Identity & Access Context

**Rationale:**
- Multiple downstream contexts depend on it
- Provides well-defined, stable API
- Published language: User ID, Email, Role

**API Characteristics:**
- RESTful, versioned (/v1/users)
- OpenAPI specification
- Backward compatible changes only
- Deprecation policy: 6 months notice

---

#### 6.1.6 Published Language

**Applied To:** Domain Events (Kafka)

**Format:** JSON Schema

**Example:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "UserRegistered",
  "type": "object",
  "properties": {
    "event_id": {"type": "string", "format": "uuid"},
    "event_type": {"type": "string", "const": "UserRegistered"},
    "timestamp": {"type": "string", "format": "date-time"},
    "version": {"type": "string", "const": "1.0"},
    "payload": {
      "type": "object",
      "properties": {
        "user_id": {"type": "string", "format": "uuid"},
        "email": {"type": "string", "format": "email"},
        "role": {"type": "string", "enum": ["labeler", "reviewer", "admin", "consumer"]}
      },
      "required": ["user_id", "email", "role"]
    }
  },
  "required": ["event_id", "event_type", "timestamp", "version", "payload"]
}
```

**Governance:**
- Schema registry (Confluent Schema Registry)
- Versioning: Semantic versioning
- Breaking changes: New event type or version

---

### 6.2 Pattern Decision Matrix

| Pattern | When to Use | Benefits | Risks |
|---------|-------------|----------|-------|
| **Shared Kernel** | Same team, high cohesion | Low overhead | Tight coupling |
| **Customer/Supplier** | Clear customer needs | Customer-driven API | Requires coordination |
| **Conformist** | Upstream is stable, trusted | Simple integration | Coupling to upstream |
| **ACL** | External systems, unstable upstream | Isolation from changes | Additional complexity |
| **OHS** | Multiple consumers | Stable, reusable API | Maintenance burden |
| **Published Language** | Async integration | Decoupling | Schema evolution complexity |

---

## 7. Domain Model

### 7.1 Core Domain: Quality Assurance

#### Aggregate: QualityEvaluation

```
┌─────────────────────────────────────────────────────────────┐
│                    QualityEvaluation                         │
│                   (Aggregate Root)                           │
├─────────────────────────────────────────────────────────────┤
│ + id: QualityEvaluationId                                   │
│ + batch_id: BatchId                                         │
│ + status: EvaluationStatus                                  │
│ + created_at: Timestamp                                     │
├─────────────────────────────────────────────────────────────┤
│ + evaluate(): QualityScore                                  │
│ + require_review(): void                                    │
│ + accept(): void                                            │
│ + reject(reason: string): void                              │
└─────────────────────────────────────────────────────────────┘
                        │
                        │ contains
                        ▼
        ┌───────────────────────────────┐
        │      QualityScore             │
        │     (Value Object)            │
        ├───────────────────────────────┤
        │ + wer_before: decimal         │
        │ + wer_after: decimal          │
        │ + agreement: decimal          │
        │ + overall: decimal            │
        ├───────────────────────────────┤
        │ + is_acceptable(): boolean    │
        │ + improvement(): decimal      │
        └───────────────────────────────┘

                        │
                        │ contains
                        ▼
        ┌───────────────────────────────┐
        │         Review                │
        │        (Entity)               │
        ├───────────────────────────────┤
        │ + id: ReviewId                │
        │ + reviewer_id: UserId         │
        │ + decision: ReviewDecision    │
        │ + notes: string               │
        │ + reviewed_at: Timestamp      │
        └───────────────────────────────┘
```

**Invariants:**
1. `overall_score = (1 - wer_after) * 0.6 + agreement * 0.4`
2. `status` transitions: `pending → evaluated → {accepted | review_required}`
3. Reviews can only be created when `status == review_required`

**Business Rules:**
- If `overall_score < 0.8`, require review
- If `agreement < 0.7`, require review
- If anomaly detected, high-priority review
- Maximum 3 review levels (labeler → reviewer → admin)

---

### 7.2 Core Domain: Task Assignment

#### Aggregate: Batch

```
┌─────────────────────────────────────────────────────────────┐
│                         Batch                                │
│                   (Aggregate Root)                           │
├─────────────────────────────────────────────────────────────┤
│ + id: BatchId                                               │
│ + name: string                                              │
│ + priority: Priority                                        │
│ + status: BatchStatus                                       │
│ + deadline: Timestamp?                                      │
│ + created_at: Timestamp                                     │
├─────────────────────────────────────────────────────────────┤
│ + assign_to(labeler: LabelerId): Assignment                │
│ + lock(duration: Duration): void                            │
│ + unlock(): void                                            │
│ + submit(): void                                            │
│ + calculate_priority(): Priority                            │
└─────────────────────────────────────────────────────────────┘
                        │
                        │ contains
                        ▼
        ┌───────────────────────────────┐
        │       Assignment              │
        │        (Entity)               │
        ├───────────────────────────────┤
        │ + id: AssignmentId            │
        │ + labeler_id: LabelerId       │
        │ + assigned_at: Timestamp      │
        │ + locked_until: Timestamp     │
        │ + submitted_at: Timestamp?    │
        ├───────────────────────────────┤
        │ + is_locked(): boolean        │
        │ + is_expired(): boolean       │
        └───────────────────────────────┘

                        │
                        │ contains
                        ▼
        ┌───────────────────────────────┐
        │         Priority              │
        │     (Value Object)            │
        ├───────────────────────────────┤
        │ + score: integer              │
        │ + tier: QualityTier           │
        │ + deadline_urgency: integer   │
        │ + difficulty: Difficulty      │
        ├───────────────────────────────┤
        │ + calculate(): integer        │
        └───────────────────────────────┘
```

**Invariants:**
1. Batch can only have one active assignment at a time
2. Lock must expire after TTL (2-4 hours)
3. Cannot submit if not assigned
4. Priority recalculated when deadline or tier changes

**Business Rules:**
- `priority_score = tier_weight * 50 + deadline_urgency * 30 + difficulty * 20`
- Auto-unlock if `locked_until < now()` and not submitted
- Notify labeler when high-priority batch matches skills

---

### 7.3 Supporting Domain: User Profile

#### Aggregate: Labeler

```
┌─────────────────────────────────────────────────────────────┐
│                        Labeler                               │
│                   (Aggregate Root)                           │
├─────────────────────────────────────────────────────────────┤
│ + user_id: UserId                                           │
│ + profile: Profile                                          │
│ + rating: Rating                                            │
│ + status: LabelerStatus                                     │
├─────────────────────────────────────────────────────────────┤
│ + add_skill(skill: Skill): void                             │
│ + verify_skill(skill: Skill, verifier: UserId): void        │
│ + update_rating(new_rating: integer, reason: string): void  │
│ + matches_requirements(requirements: SkillSet): boolean     │
└─────────────────────────────────────────────────────────────┘
                        │
                        │ contains
                        ▼
        ┌───────────────────────────────┐
        │          Skill                │
        │     (Value Object)            │
        ├───────────────────────────────┤
        │ + type: SkillType             │
        │ + value: string               │
        │ + proficiency: Proficiency    │
        │ + verified: boolean           │
        ├───────────────────────────────┤
        │ + matches(requirement): bool  │
        └───────────────────────────────┘

                        │
                        │ contains
                        ▼
        ┌───────────────────────────────┐
        │         Rating                │
        │        (Entity)               │
        ├───────────────────────────────┤
        │ + current: integer            │
        │ + history: RatingChange[]     │
        ├───────────────────────────────┤
        │ + update(new, reason): void   │
        │ + calculate_elo(...): integer │
        └───────────────────────────────┘
```

**Invariants:**
1. Rating must be between 0 and 3000
2. Skills must be verified before used for routing
3. Cannot have duplicate skills (same type + value)

**Business Rules:**
- New labelers start with rating 1000
- Elo K-factor = 32 for rating < 2000, 24 for rating >= 2000
- Minimum 10 batches before rating is considered stable

---

## 8. Architecture Decision Records

### ADR-001: Microservices per Bounded Context

**Status:** Accepted

**Context:**
We need to decide on the service granularity for the ADLP platform.

**Decision:**
We will implement **one microservice per Bounded Context**, resulting in 9 microservices:
1. Identity & Access Service
2. User Profile Service
3. Wallet & Payment Service
4. Data Ingestion Service
5. Prelabeling Service
6. Task Assignment Service
7. Labeling Service
8. Quality Assurance Service
9. Dataset Export Service

**Rationale:**
- Aligns with DDD principle: Bounded Context = Deployment Unit
- Enables independent scaling and deployment
- Clear ownership per team
- Reduces coupling between domains

**Consequences:**
- **Positive:** Clear boundaries, independent deployment, team autonomy
- **Negative:** Increased operational complexity, distributed transactions
- **Mitigation:** Event-driven architecture, saga pattern for workflows

---

### ADR-002: Event-Driven Integration

**Status:** Accepted

**Context:**
Bounded Contexts need to communicate. We need to choose between synchronous (REST) and asynchronous (events) integration.

**Decision:**
Use **Event-Driven Architecture** with Kafka as the primary integration mechanism, with REST for synchronous queries.

**Rationale:**
- Decouples contexts (temporal and spatial)
- Enables eventual consistency
- Supports event sourcing and audit trails
- Aligns with reactive systems principles

**Consequences:**
- **Positive:** Loose coupling, scalability, resilience
- **Negative:** Eventual consistency, debugging complexity
- **Mitigation:** Event schema registry, distributed tracing, saga orchestration

---

### ADR-003: No Shared Database

**Status:** Accepted

**Context:**
Multiple services need to access user data, batch data, etc.

**Decision:**
Each Bounded Context owns its **private database**. No shared database across contexts.

**Rationale:**
- Enforces bounded context boundaries
- Prevents coupling through database
- Enables independent schema evolution
- Aligns with microservices best practices

**Consequences:**
- **Positive:** Strong encapsulation, independent evolution
- **Negative:** Data duplication, eventual consistency
- **Mitigation:** CQRS for read models, event-driven synchronization

---

### ADR-004: Aggregate Design

**Status:** Accepted

**Context:**
We need to define transaction boundaries within each Bounded Context.

**Decision:**
Use **Aggregates** as transaction boundaries. Each Aggregate has:
- One Aggregate Root (entity)
- Internal entities and value objects
- Invariants enforced within aggregate
- Changes published as Domain Events

**Rationale:**
- Ensures consistency within transaction boundary
- Limits transaction scope for performance
- Aligns with DDD tactical patterns

**Consequences:**
- **Positive:** Clear consistency boundaries, better performance
- **Negative:** Eventual consistency across aggregates
- **Mitigation:** Saga pattern for cross-aggregate workflows

---

### ADR-005: Domain Events as First-Class Citizens

**Status:** Accepted

**Context:**
We need a mechanism for cross-context communication and audit trails.

**Decision:**
Model **Domain Events** explicitly as first-class domain concepts. All state changes publish events.

**Rationale:**
- Enables event sourcing
- Provides audit trail
- Supports CQRS
- Enables reactive workflows

**Event Schema:**
```json
{
  "event_id": "uuid",
  "event_type": "DomainEventName",
  "timestamp": "ISO8601",
  "version": "semver",
  "correlation_id": "uuid",
  "causation_id": "uuid",
  "payload": { }
}
```

**Consequences:**
- **Positive:** Full audit trail, temporal queries, event replay
- **Negative:** Storage overhead, schema evolution complexity
- **Mitigation:** Event versioning strategy, schema registry

---

### ADR-006: Anti-Corruption Layer for External Systems

**Status:** Accepted

**Context:**
We integrate with external payment providers (Stripe, PayPal) and AI models (SageMaker).

**Decision:**
Implement **Anti-Corruption Layers (ACL)** to isolate domain from external models.

**Rationale:**
- Protects domain model from external changes
- Translates external concepts to domain language
- Enables switching providers without domain changes

**Consequences:**
- **Positive:** Domain model stability, provider independence
- **Negative:** Additional translation layer
- **Mitigation:** Keep ACL thin, test thoroughly

---


### ADR-007: Service Level Objectives (SLOs) by Context

**Status:** Accepted

**Context:**
Different Bounded Contexts have different criticality and performance requirements. We need clear SLOs to guide architecture and operational decisions.

**Decision:**
Define SLOs per context based on business impact and user expectations.

---

#### SLO Tier Classification

| Tier | Availability | Latency (p95) | Error Budget | Examples |
|------|--------------|---------------|--------------|----------|
| **Critical** | 99.95% | <200ms | 0.05% | Identity & Access, Task Assignment |
| **High** | 99.9% | <500ms | 0.1% | User Profile, Labeling, Quality |
| **Standard** | 99.5% | <1s | 0.5% | Wallet, Data Ingestion, Prelabeling |
| **Low** | 99% | <5s | 1% | Dataset Export |

---

#### Context-Specific SLOs

**BC1: Identity & Access Context**

| Metric | SLO | Measurement | Consequences of Breach |
|--------|-----|-------------|------------------------|
| **Availability** | 99.95% | Uptime monitoring | Users cannot login, entire platform unusable |
| **Latency (p95)** | <100ms | API response time | Poor user experience, timeout errors |
| **Latency (p99)** | <200ms | API response time | Cascading delays in dependent services |
| **Error Rate** | <0.1% | Failed requests / total | Authentication failures, security incidents |
| **Token Refresh** | <50ms | JWT refresh time | Session timeouts, user frustration |

**Rationale:** Authentication is on critical path for all operations.

---

**BC2: User Profile Context**

| Metric | SLO | Measurement | Consequences of Breach |
|--------|-----|-------------|------------------------|
| **Availability** | 99.9% | Uptime monitoring | Cannot route batches, assignment fails |
| **Latency (p95)** | <300ms | Search API response | Slow batch assignment |
| **Latency (p99)** | <500ms | Search API response | Assignment timeouts |
| **Error Rate** | <0.2% | Failed requests / total | Incorrect routing, poor labeler matching |
| **Rating Update** | <1s | Update confirmation | Delayed feedback to labelers |

**Rationale:** Supports core routing logic, but not on critical user path.

---

**BC3: Wallet & Payment Context**

| Metric | SLO | Measurement | Consequences of Breach |
|--------|-----|-------------|------------------------|
| **Availability** | 99.5% | Uptime monitoring | Cannot credit earnings, withdrawal delays |
| **Latency (p95)** | <500ms | Credit/debit operations | Slow payout processing |
| **Data Consistency** | 100% | Transaction integrity | Financial loss, compliance violations |
| **Audit Trail** | 100% | Log completeness | Regulatory non-compliance |
| **Withdrawal Processing** | <24h | Time to complete | Labeler dissatisfaction |

**Rationale:** Financial integrity is critical, but async operations tolerate higher latency.

---

**BC5: Prelabeling Context**

| Metric | SLO | Measurement | Consequences of Breach |
|--------|-----|-------------|------------------------|
| **Availability** | 99.5% | Uptime monitoring | Prelabeling backlog, delayed batches |
| **Throughput** | >10 segments/s | Processing rate | Cannot keep up with ingestion |
| **Latency (p95)** | <10s | Inference time | Slow prelabeling pipeline |
| **Model Accuracy** | >85% WER | Quality metrics | Poor prelabels, more manual work |
| **Queue Depth** | <1000 items | Kafka lag | Backlog indicates scaling needed |

**Rationale:** Async workflow, higher latency acceptable, but throughput critical.

---

**BC6: Task Assignment Context**

| Metric | SLO | Measurement | Consequences of Breach |
|--------|-----|-------------|------------------------|
| **Availability** | 99.95% | Uptime monitoring | Labelers cannot get work, revenue loss |
| **Latency (p95)** | <200ms | Batch assignment | Slow labeler experience |
| **Routing Accuracy** | >90% | Correct skill match | Poor quality, labeler frustration |
| **Lock Expiry** | 100% | TTL enforcement | Batches stuck, workflow blocked |
| **Priority Calculation** | <100ms | Calculation time | Incorrect prioritization |

**Rationale:** Core domain, on critical path for labeler workflow.

---

**BC8: Quality Assurance Context**

| Metric | SLO | Measurement | Consequences of Breach |
|--------|-----|-------------|------------------------|
| **Availability** | 99.9% | Uptime monitoring | Quality backlog, delayed payouts |
| **Processing Time** | <5min per batch | Evaluation duration | Slow feedback to labelers |
| **Accuracy** | >95% | Quality score correctness | Incorrect ratings, unfair payouts |
| **Review SLA** | <4h | Time to review | Labeler frustration, workflow delays |
| **Bias Detection** | 100% coverage | Datasets scanned | Compliance risk, unfair datasets |

**Rationale:** Core domain, but async processing allows higher latency.

---

#### Event Processing SLOs

| Event Type | Max Latency | Retry Policy | DLQ Threshold |
|------------|-------------|--------------|---------------|
| **UserRegistered** | <1s | 3 retries, exponential backoff | After 3 failures |
| **PrelabelCompleted** | <10s | 5 retries, exponential backoff | After 5 failures |
| **BatchSubmitted** | <30s | 3 retries, exponential backoff | After 3 failures |
| **QualityEvaluated** | <5min | 5 retries, exponential backoff | After 5 failures |
| **EarningCredited** | <1min | 10 retries, exponential backoff | Manual intervention |

**Rationale:**
- Financial events (EarningCredited) have aggressive retries
- Workflow events tolerate higher latency
- All events eventually go to DLQ for manual review

---

#### Data Retention & Lifecycle

| Data Type | Retention (Hot) | Retention (Cold) | Deletion Policy |
|-----------|-----------------|------------------|-----------------|
| **Raw Audio** | 90 days (S3 Standard) | 2 years (Glacier) | After 2 years or user request |
| **Segments** | 180 days (S3 Standard) | N/A | After dataset export |
| **Transcripts** | 1 year (Postgres) | 3 years (Archive DB) | After 3 years |
| **Quality Scores** | 1 year (Postgres) | 5 years (Archive DB) | After 5 years (compliance) |
| **Audit Logs** | 90 days (CloudWatch) | 7 years (S3 Glacier) | After 7 years (legal requirement) |
| **Domain Events** | 30 days (Kafka) | 1 year (S3) | After 1 year |
| **Wallet Transactions** | Indefinite (Postgres) | N/A | Never (financial records) |

---

#### Disaster Recovery SLOs

| Context | RPO (Recovery Point Objective) | RTO (Recovery Time Objective) | Backup Strategy |
|---------|--------------------------------|-------------------------------|-----------------|
| **Identity & Access** | <5 minutes | <15 minutes | Multi-AZ RDS, automated snapshots |
| **User Profile** | <15 minutes | <30 minutes | Multi-AZ RDS, automated snapshots |
| **Wallet & Payment** | <1 minute | <10 minutes | Multi-AZ RDS, transaction log shipping |
| **Quality Assurance** | <1 hour | <2 hours | Daily snapshots, event replay |
| **Prelabeling** | <4 hours | <8 hours | Event replay from Kafka |

**Rationale:**
- Financial data (Wallet) has strictest requirements
- Core domains (Quality, Assignment) have moderate requirements
- Supporting domains can tolerate longer recovery times

---

## SECTION 10.5: Enterprise Governance (INSERT AT END)

## 9. Implementation Roadmap

### 9.1 Phase 1: Foundation (Weeks 1-4)

**Objective:** Establish core infrastructure and generic domains

**Deliverables:**
- [ ] Identity & Access Service (BC1)
  - User registration, login, JWT
  - RBAC enforcement
  - Email verification
- [ ] Infrastructure setup
  - Kafka cluster
  - PostgreSQL per service
  - Redis for caching
  - S3 buckets
- [ ] CI/CD pipeline
  - GitHub Actions
  - ECS deployment
  - Smoke tests

**Team:** Security Team + DevOps

---

### 9.2 Phase 2: User Management (Weeks 5-8)

**Objective:** Build supporting domains for user management

**Deliverables:**
- [ ] User Profile Service (BC2)
  - Profile CRUD
  - Skills management
  - Rating tracking
  - Search API
- [ ] Wallet & Payment Service (BC3)
  - Wallet creation
  - Transaction recording
  - Withdrawal workflow
  - Stripe integration (ACL)

**Team:** User Management Team + Finance Team

---

### 9.3 Phase 3: Data Pipeline (Weeks 9-12)

**Objective:** Build data ingestion and prelabeling

**Deliverables:**
- [ ] Data Ingestion Service (BC4)
  - Upload API
  - Audio normalization
  - Deduplication
  - Metadata extraction
- [ ] Prelabeling Service (BC5)
  - SageMaker integration
  - Model routing
  - Segmentation
  - Confidence scoring

**Team:** Data Engineering Team + ML Engineering Team

---

### 9.4 Phase 4: Core Workflow (Weeks 13-16)

**Objective:** Build core labeling workflow

**Deliverables:**
- [ ] Task Assignment Service (BC6)
  - Batch creation
  - Priority calculation
  - Skill-based routing
  - Lock management
- [ ] Labeling Service (BC7)
  - Labeling UI backend
  - Transcript editing
  - Autosave
  - Submission

**Team:** Workflow Team + Frontend Team

---

### 9.5 Phase 5: Quality & Export (Weeks 17-20)

**Objective:** Build quality assurance and export

**Deliverables:**
- [ ] Quality Assurance Service (BC8)
  - WER calculation
  - Overlap sampling
  - Automated review
  - Bias detection
- [ ] Dataset Export Service (BC9)
  - Dataset compilation
  - Format conversion
  - Signed URL generation

**Team:** Quality Team + Data Engineering Team

---

### 9.6 Phase 6: Integration & Testing (Weeks 21-24)

**Objective:** End-to-end integration and testing

**Deliverables:**
- [ ] Integration testing
  - Cross-context workflows
  - Event-driven flows
  - Error scenarios
- [ ] Performance testing
  - Load testing (1000 concurrent labelers)
  - Latency benchmarks
  - Scalability validation
- [ ] Security testing
  - Penetration testing
  - RBAC validation
  - Data encryption verification
- [ ] UAT with pilot users

**Team:** All Teams + QA

---

## 10. Compliance & Standards

### 10.1 ISO/IEC/IEEE 42010:2011 Compliance

This document complies with **ISO/IEC/IEEE 42010:2011** (Systems and software engineering — Architecture description):

| Requirement | Section | Compliance |
|-------------|---------|------------|
| **Architecture Description** | Entire document | ✅ Complete |
| **Stakeholders & Concerns** | Section 1.3 | ✅ Identified |
| **Architecture Viewpoints** | Sections 3-7 | ✅ Domain, Context, Integration views |
| **Architecture Views** | Sections 3-7 | ✅ Context Map, Domain Models |
| **Architecture Rationale** | Section 8 (ADRs) | ✅ Documented |
| **Correspondence Rules** | Section 4.3 | ✅ Integration patterns |

### 10.2 DDD Best Practices Compliance

| Practice | Source | Compliance |
|----------|--------|------------|
| **Bounded Context per Team** | Eric Evans, DDD | ✅ 9 contexts, clear ownership |
| **Ubiquitous Language** | Eric Evans, DDD | ✅ Section 5 |
| **Context Mapping** | Eric Evans, DDD | ✅ Section 4 |
| **Strategic Patterns** | Vaughn Vernon, IDDD | ✅ Section 6 |
| **Aggregate Design** | Vaughn Vernon, IDDD | ✅ Section 7 |
| **Domain Events** | Vaughn Vernon, IDDD | ✅ Sections 5.2, ADR-005 |
| **Anti-Corruption Layer** | Eric Evans, DDD | ✅ ADR-006 |

### 10.3 Microservices Best Practices

| Practice | Source | Compliance |
|----------|--------|------------|
| **Database per Service** | Sam Newman, Building Microservices | ✅ ADR-003 |
| **Event-Driven Integration** | Chris Richardson, Microservices Patterns | ✅ ADR-002 |
| **API Gateway** | Chris Richardson | ✅ Gateway Service |
| **Circuit Breaker** | Michael Nygard, Release It! | ✅ Planned in tactical design |
| **Distributed Tracing** | Cindy Sridharan, Distributed Systems Observability | ✅ OpenTelemetry |

### 10.4 Security & Privacy Standards

| Standard | Requirement | Compliance |
|----------|-------------|------------|
| **GDPR** | Data protection, right to deletion | ✅ User consent, data deletion APIs |
| **ISO 27001** | Information security management | ✅ Encryption, access control, audit logs |
| **OWASP Top 10** | Web application security | ✅ JWT, input validation, SQL injection prevention |
| **PCI DSS** | Payment card security | ✅ ACL for payment providers, no card storage |

---

## Appendix A: Glossary

## SECTION 10.5: Enterprise Governance (INSERT AT END)

### 10.5 Enterprise Governance & Quality Assurance

#### 10.5.1 Architecture Review Board (ARB)

**Purpose:** Ensure architectural decisions align with strategic design and enterprise standards.

**Composition:**
- Chief Architect (Chair)
- Domain Architects (1 per Core Domain)
- Security Architect
- Data Architect
- Representative from each team

**Review Triggers:**
- New Bounded Context proposal
- Change to Context Map (integration pattern)
- Major technology change (e.g., new database, message broker)
- SLA changes
- Security or compliance concerns

**Review Criteria:**
- [ ] Aligns with DDD strategic design
- [ ] Follows established patterns
- [ ] Has clear ownership
- [ ] Documented ADR with rationale
- [ ] Security reviewed
- [ ] Performance impact assessed
- [ ] Operational readiness (monitoring, runbooks)

**Cadence:** Bi-weekly + ad-hoc for urgent decisions

---

#### 10.5.2 Tactical Design Standards

**Per Bounded Context, teams must deliver:**

1. **Tactical DDD Design Document**
   - Aggregates with invariants
   - Entities and Value Objects
   - Domain Services
   - Repositories
   - Domain Events

2. **API Specification**
   - OpenAPI 3.0 for REST
   - AsyncAPI for events
   - Versioning strategy
   - Deprecation policy

3. **Database Schema**
   - ER diagram
   - Migration scripts (versioned)
   - Indexing strategy
   - Partitioning plan (if applicable)

4. **Testing Strategy**
   - Unit tests (>80% coverage for domain logic)
   - Integration tests (API contracts)
   - Event contract tests
   - Performance tests (load, stress)

5. **Operational Readiness**
   - Runbook (common issues, resolutions)
   - Monitoring dashboard
   - Alert definitions
   - Disaster recovery plan

---

#### 10.5.3 Continuous Compliance

**Quarterly Reviews:**
- [ ] Context boundaries still valid?
- [ ] Integration patterns still appropriate?
- [ ] SLOs being met?
- [ ] Technical debt assessment
- [ ] Security audit
- [ ] Cost optimization review

**Annual Strategic Review:**
- [ ] Domain model still reflects business?
- [ ] Core/Supporting/Generic classification still correct?
- [ ] Team structure aligns with contexts?
- [ ] Technology stack still appropriate?
- [ ] Competitive landscape changes?

---

#### 10.5.4 Knowledge Management

**Documentation Repository:**
```
docs/
├── strategic/
│   ├── DDD_STRATEGIC_DESIGN.md (this document)
│   ├── ARCHITECTURE_GAP_ANALYSIS.md
│   └── SYSTEM_ARCHITECTURE_COMPREHENSIVE.md
├── tactical/
│   ├── identity-access/
│   │   ├── tactical-design.md
│   │   ├── api-spec.yaml
│   │   └── schema.sql
│   ├── user-profile/
│   │   └── ...
│   └── ...
├── adrs/
│   ├── ADR-001-microservices-per-context.md
│   ├── ADR-002-event-driven-integration.md
│   └── ...
├── runbooks/
│   ├── identity-access-runbook.md
│   ├── incident-response.md
│   └── ...
└── onboarding/
    ├── ddd-primer.md
    ├── codebase-tour.md
    └── development-workflow.md
```

**Training Program:**
- DDD fundamentals (2-day workshop)
- Tactical patterns (1-day workshop)
- Event Storming facilitation (1-day workshop)
- Code reviews focused on domain modeling
- Pair programming with domain experts

---

## INTEGRATION INSTRUCTIONS

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Aggregate** | A cluster of domain objects treated as a single unit for data changes |
| **Aggregate Root** | The entry point entity of an aggregate, enforcing invariants |
| **Bounded Context** | An explicit boundary within which a domain model is defined |
| **Context Map** | A diagram showing relationships between bounded contexts |
| **Domain Event** | A record of something that happened in the domain |
| **Entity** | An object with identity that persists over time |
| **Invariant** | A business rule that must always be true |
| **Ubiquitous Language** | A common language shared by developers and domain experts |
| **Value Object** | An immutable object defined by its attributes, not identity |

---

## Appendix B: References

1. **Eric Evans** - *Domain-Driven Design: Tackling Complexity in the Heart of Software* (2003)
2. **Vaughn Vernon** - *Implementing Domain-Driven Design* (2013)
3. **Vaughn Vernon** - *Domain-Driven Design Distilled* (2016)
4. **Sam Newman** - *Building Microservices* (2nd Edition, 2021)
5. **Chris Richardson** - *Microservices Patterns* (2018)
6. **ISO/IEC/IEEE 42010:2011** - Systems and software engineering — Architecture description
7. **Martin Fowler** - *Patterns of Enterprise Application Architecture* (2002)
8. **Gregor Hohpe, Bobby Woolf** - *Enterprise Integration Patterns* (2003)

---

## Appendix C: Review Checklist

### Strategic Design Review

- [ ] All subdomains classified (Core, Supporting, Generic)
- [ ] Bounded Contexts identified with clear boundaries
- [ ] Context Map shows all relationships
- [ ] Integration patterns selected and justified
- [ ] Ubiquitous Language defined per context
- [ ] Aggregates identified with invariants
- [ ] Domain Events modeled
- [ ] ADRs document key decisions
- [ ] Implementation roadmap defined
- [ ] Compliance verified (ISO 42010, DDD, security)

### Stakeholder Sign-off

- [ ] Domain Experts: Model reflects business reality
- [ ] Architects: Structure is sound and maintainable
- [ ] Developers: Boundaries are clear and implementable
- [ ] Product Owners: Roadmap delivers business value
- [ ] Security: Compliance requirements met
- [ ] Operations: System is deployable and observable

---

**Document Status:** Ready for Review  
**Next Steps:**
1. Stakeholder review (1 week)
2. Incorporate feedback
3. Approval and sign-off
4. Proceed to Tactical Design (per Bounded Context)

---

**END OF DOCUMENT**
