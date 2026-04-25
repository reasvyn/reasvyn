# Standardized Operational Manual for AI Agentic Workflows
### ISO/IEC Compliant Software Engineering Governance Framework

---

**Document Reference:** ISO-IEC-12207-AGENT-2026
**Revision:** 3.0.0
**Compliance Standards:**
- ISO/IEC/IEEE 12207:2017 — Software Life Cycle Processes
- ISO/IEC 25010:2011 — System and Software Quality Models

**Intended Consumers:** AI Agents, Autonomous Coding Assistants, Agentic Workflow Engines
**Scope:** Engineering governance, architectural control, and behavioral constraints for any AI agent operating in a software development lifecycle context.

---

## PREFACE — How to Read This Document

This document is structured for deterministic machine consumption. Each section is a **self-contained control unit** with:

- A **normative statement** (what MUST happen),
- A **rationale** (why it is required), and
- **Violation conditions** (what constitutes non-compliance).

**Priority Resolution:** When provisions conflict, resolve using the following hierarchy:

```
SECURITY (S1) > SUSTAINABILITY (S2) > SCALABILITY (S3)
```

**Keyword Semantics (RFC 2119):**
- `SHALL` / `MUST` — Unconditional requirement. Non-compliance is a governance failure.
- `SHALL NOT` / `MUST NOT` — Absolute prohibition.
- `SHOULD` — Strong recommendation; deviation requires explicit justification.
- `MAY` — Optional; permitted but not required.

---

## GOVERNING DOCTRINE — The 3S Principles

All engineering decisions, architectural changes, and agent behaviors SHALL be evaluated against the 3S doctrine. No provision in this document may weaken these principles.

| Principle | Code | Definition |
|-----------|------|------------|
| **Secure** | S1 | Preservation of integrity, boundary contracts, failure semantics, and behavioral predictability. |
| **Sustain** | S2 | Maintainability, semantic clarity, documentation completeness, and reduced cognitive load for both humans and agents. |
| **Scalable** | S3 | Structural modularity, controlled dependency growth, and predictable performance under system expansion. |

> **Conflict Rule:** When two principles are in tension, `S1 > S2 > S3`. Security is non-negotiable.

---

## PART I — ARCHITECTURAL GOVERNANCE

---

### Section 1 — Separation of Concerns (SOC)

#### 1.1 Mandate

Separation of Concerns is enforced as a structural compliance requirement under ISO/IEC 25010:2011 Maintainability and Modularity characteristics.

#### 1.2 Layered Architecture Model

The system SHALL maintain strict logical isolation across the following layers. Cross-layer coupling MUST occur exclusively through explicit, typed abstractions.

```
┌──────────────────────────────────────────────────────┐
│  INTERFACE LAYER                                      │
│  Delivery mechanisms: HTTP controllers, CLI adapters, │
│  UI endpoints, event consumers                        │
│  ↕ (via explicit adapter contracts only)             │
├──────────────────────────────────────────────────────┤
│  APPLICATION LAYER                                    │
│  Use-case orchestration, transaction coordination,    │
│  workflow sequencing                                  │
│  ↕ (via domain interfaces only)                      │
├──────────────────────────────────────────────────────┤
│  DOMAIN LAYER                                         │
│  Business rules, entities, value objects, invariants │
│  NO infrastructure, framework, or I/O dependencies   │
│  ↕ (via repository/port interfaces only)             │
├──────────────────────────────────────────────────────┤
│  INFRASTRUCTURE LAYER                                 │
│  External integrations: databases, file systems,     │
│  APIs, messaging, third-party frameworks             │
└──────────────────────────────────────────────────────┘
```

#### 1.3 Violations

An SOC violation is declared when any of the following occur:

- Domain layer contains a direct import of an infrastructure library (e.g., ORM, HTTP client, file system).
- Application layer bypasses domain interfaces to access infrastructure directly.
- Interface layer contains business logic.
- Non-testable isolation exists at any layer boundary.

#### 1.4 Compliance Criterion

Each concern SHALL be independently testable:
- Domain logic: testable without infrastructure.
- Application services: testable via mocked abstractions.
- Infrastructure adapters: testable in isolation.

**Non-testable isolation is treated as an SOC violation.**

---

### Section 2 — Dependency Governance

#### 2.1 Inversion Mandate

All modules SHALL depend on abstractions, not on concrete implementations. Dependency inversion is mandatory throughout the codebase.

#### 2.2 Prohibited Patterns

| Pattern | Classification |
|---------|---------------|
| Cross-module reference to a concrete class | Architectural violation |
| Undocumented dependency expansion | Architectural violation |
| Transitive dependency chain across boundaries | Structural risk |
| Implicit shared mutable state across modules | S1 Security violation |

#### 2.3 Permitted Exception

Concrete classes MAY be referenced only if **explicitly designated as stable boundary constructs**, defined as one of:
- Façade
- Contract adapter
- Framework bridge

This designation MUST be documented in the architecture decision record.

---

### Section 3 — Boundary Contract Specification

#### 3.1 Requirements

All architectural boundaries MUST declare:

1. **Explicit typed interfaces** — No implicit duck typing or runtime-discovered contracts.
2. **Input/output invariants** — Preconditions and postconditions MUST be documented.
3. **Absence of implicit shared state** — All shared state MUST be passed explicitly.
4. **Freedom from transitive dependency chains** — A boundary must not silently pull in unrelated dependencies.

#### 3.2 Boundary Ambiguity

Boundary ambiguity is classified as **structural risk (S1)**. Ambiguous boundaries MUST be resolved before any implementation proceeds. The agent SHALL NOT proceed with implementation when boundary ambiguity exists.

---

## PART II — CODE QUALITY STANDARDS

---

### Section 4 — Clean Code Standard (Maintainability Control Layer)

Clean Code is an enforceable maintainability discipline under the Sustain (S2) principle.

#### 4.1 Naming and Semantic Clarity

| Rule | Requirement |
|------|-------------|
| Identifier naming | SHALL express intent, not implementation detail |
| Generic/ambiguous names | Prohibited unless contextually justified and documented |
| Domain terminology | MUST remain consistent and uniform across all modules |
| Abbreviations | Permitted only for universally established acronyms (e.g., `id`, `url`, `dto`) |

#### 4.2 Function Design

- Functions SHALL implement **exactly one responsibility**.
- A function that requires an "and" in its description violates the Single Responsibility Principle and MUST be decomposed.
- Parameter lists SHALL remain minimal. More than 3–4 parameters is a signal to introduce a parameter object.
- Hidden dependencies (e.g., global state access, static singletons inside functions) are **prohibited**.

#### 4.3 Class Design

- Classes SHALL represent **a single axis of change**.
- Cohesion MUST be high; coupling MUST be low.
- Classes that grow beyond a single responsibility MUST be split.

#### 4.4 Code Hygiene Rules

| Item | Rule |
|------|------|
| Dead code | SHALL be removed immediately |
| Redundant branching | SHALL be eliminated |
| Side effects | SHALL be explicit, documented, and isolated |
| Formatting | SHALL be consistent and enforced by automated tooling |
| Magic numbers/strings | MUST be extracted into named constants or configuration |
| Hardcoded values | MUST NOT appear in logic; use central configuration repositories |

> **Rationale:** Readability is treated as an architectural constraint, not a stylistic preference. Code is read far more often than it is written.

---

### Section 5 — DRY Principle (Scalability Control Layer)

**DRY — Don't Repeat Yourself.** Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

#### 5.1 Prohibited Duplication

The following SHALL NOT be duplicated anywhere in the codebase:

- Business rules and domain logic
- Validation logic
- Constant and configuration definitions
- Data transformation and mapping logic
- Error message strings

#### 5.2 Single Source of Truth (SSOT)

Every invariant, rule, or constant SHALL:
1. Be **defined exactly once**.
2. Be **referenced, not replicated**.
3. Be **centrally modifiable** without cascading edits.

#### 5.3 Abstraction Discipline

DRY enforcement SHALL NOT result in:

- **Premature generalization** — Abstraction must be earned by actual, repeated use.
- **Clarity reduction** — An abstraction that obscures intent is worse than duplication.
- **SOC violations** — Shared abstractions MUST not blur layer boundaries.

> **Key Principle:** Abstraction is justified by **semantic identity**, not superficial structural similarity. Two loops that look similar but represent fundamentally different operations should NOT be unified.

---

## PART III — DOMAIN-DRIVEN DESIGN (DDD) GOVERNANCE

Domain-Driven Design is mandated as the **primary architectural derivation and structural organization methodology**. DDD serves as the primary enforcement mechanism for Sections 1–5, ensuring that the software model is a faithful, evolving reflection of the business domain.

---

### Section 6 — Modular DDD Architecture

#### 6.1 Bounded Context as the Primary Modular Unit

A **Bounded Context** is the central organizing principle of the entire system. It defines an explicit boundary within which a specific domain model is valid, consistent, and internally coherent.

```
┌─────────────────────────────────────────────────────────────────┐
│  BOUNDED CONTEXT — Core Properties                              │
│                                                                 │
│  • A single domain model applies EXCLUSIVELY within it.         │
│  • The Ubiquitous Language (Section 6.2) is valid ONLY here.    │
│  • All external communication MUST pass through an explicit     │
│    boundary interface (Port, ACL, or Published Language).        │
│  • A Bounded Context maps 1:1 to a deployable module or service.│
└─────────────────────────────────────────────────────────────────┘
```

**Mandatory rules for Bounded Context definition:**

| Rule | Requirement |
|------|-------------|
| Scope | Each Bounded Context SHALL encapsulate exactly one coherent subdomain |
| Isolation | No domain model object SHALL be shared across Bounded Context boundaries |
| Ownership | Each Bounded Context SHALL have a single designated owning team or module |
| Interface | Cross-context communication MUST occur through explicitly versioned contracts |

An agent MUST identify and name all relevant Bounded Contexts **before generating any domain model code**.

#### 6.2 Ubiquitous Language (UL)

The Ubiquitous Language is the shared, unambiguous vocabulary co-developed with domain experts that MUST be used uniformly across all artifacts — conversations, documentation, code identifiers, tests, and API contracts — within a given Bounded Context.

**Enforcement rules:**

- All class names, method names, field names, and event names MUST derive from the UL of their Bounded Context.
- Technical synonyms or generic terms (e.g., `Manager`, `Handler`, `Processor`, `Data`) are **prohibited** as primary identifiers where a domain-specific term exists.
- The same term MUST NOT carry different meanings within the same Bounded Context.
- When the same real-world concept exists in two Bounded Contexts with different semantics, it SHALL be represented by two distinct, context-qualified types (e.g., `billing.Customer` vs. `support.Customer`).

> **Rationale:** Code that speaks the language of the domain eliminates the translation layer between domain experts and developers, reducing the primary source of defects.

#### 6.3 Domain Model Building Blocks

All domain model elements SHALL be classified into one of the following canonical DDD building blocks. Misclassification is an architectural violation.

**6.3.1 Entity**

An object with a **persistent, unique identity** that persists across state changes.

| Property | Rule |
|----------|------|
| Identity | Defined by a unique identifier (e.g., `OrderId`, `UserId`), not by attribute values |
| Equality | Two entities are equal if and only if their identifiers are equal |
| Mutability | State MAY change; identity MUST NOT change after creation |
| Lifecycle | Has a beginning and an end within the domain |

**6.3.2 Value Object**

An immutable object whose identity is entirely defined by the **values of its attributes**.

| Property | Rule |
|----------|------|
| Immutability | MUST be immutable; no setter methods permitted |
| Equality | Two Value Objects are equal if all their attributes are equal |
| Side-effect freedom | Operations on a Value Object MUST return a new instance |
| Self-validation | SHALL enforce its own invariants at construction time (fail-fast) |

*Examples: `Money(amount, currency)`, `Address(street, city, postalCode)`, `EmailAddress(value)`*

**6.3.3 Aggregate and Aggregate Root**

An Aggregate is a **cluster of Entities and Value Objects** treated as a single transactional unit, accessed exclusively through its **Aggregate Root** (AR).

| Property | Rule |
|----------|------|
| Root access | External objects MUST reference the Aggregate exclusively via the AR |
| Invariant boundary | All business invariants within the Aggregate MUST be enforced by the AR |
| Transaction boundary | One Aggregate instance per transaction is the default rule; cross-aggregate coordination uses Domain Events |
| Identity reference | External references to internal Aggregate members MUST use identity (ID), not object reference |
| Size constraint | Aggregates SHOULD be kept small; large Aggregates are a signal of incorrect boundary definition |

**6.3.4 Domain Service**

A stateless service that encapsulates **domain logic that does not naturally belong to any single Entity or Value Object**.

- SHALL be stateless.
- SHALL operate exclusively on domain objects.
- SHALL NOT depend on infrastructure (no database calls, no HTTP, no I/O).
- Name SHALL reflect a domain action or policy (e.g., `PricingPolicy`, `FraudDetectionService`).

**6.3.5 Domain Event**

An immutable record of **something significant that happened in the domain**, expressed in the past tense from the domain's perspective.

| Property | Rule |
|----------|------|
| Naming | MUST be past-tense domain language (e.g., `OrderPlaced`, `PaymentFailed`, `InventoryDepleted`) |
| Immutability | MUST be immutable after creation |
| Content | SHALL contain enough data for consumers to react without querying back |
| Origin | SHALL be raised by an Aggregate Root as a side effect of a state-changing operation |
| Transport | Cross-context Domain Events MUST transit through explicit messaging contracts, not shared objects |

---

### Section 7 — Context Mapping and Integration Governance

#### 7.1 Context Map

A **Context Map** is a mandatory architectural document that defines the relationships between all Bounded Contexts in the system. It MUST be produced before any cross-context integration is implemented and MUST be kept current as the system evolves.

The Context Map MUST declare, for each inter-context relationship:
1. The upstream (supplier) and downstream (consumer) context.
2. The integration pattern in use (see Section 7.2).
3. The translation/anti-corruption strategy.
4. The team or module owning each side.

#### 7.2 Integration Patterns

The following canonical integration patterns SHALL be used to govern cross-context communication. Selection of a pattern MUST be documented in the Context Map.

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Shared Kernel** | Two contexts share a small, explicitly defined subset of the domain model | Teams in close collaboration; shared subset is stable and small |
| **Customer/Supplier** | Upstream publishes a contract; downstream negotiates requirements | Clear upstream/downstream team relationship |
| **Conformist** | Downstream adopts upstream's model without negotiation | Upstream has no incentive to adapt (e.g., external SaaS API) |
| **Anti-Corruption Layer (ACL)** | Downstream translates upstream's model into its own domain language | Upstream model is incompatible or would corrupt the downstream domain |
| **Open Host Service (OHS)** | Upstream publishes a formal, versioned protocol for multiple consumers | One upstream serves many downstream contexts |
| **Published Language** | A shared, well-documented exchange format (e.g., JSON Schema, Protobuf) used by OHS | Formal interoperability across organizational boundaries |
| **Separate Ways** | Contexts have no integration; each solves its problems independently | Integration cost exceeds benefit |

#### 7.3 Anti-Corruption Layer (ACL) Requirements

An ACL is **mandatory** whenever integrating with any of the following:
- An external third-party system or API.
- A legacy system with an incompatible model.
- A Bounded Context whose model would contaminate the local domain model if imported directly.

An ACL MUST:
- Translate all incoming data from the external model into local domain types before use.
- Never allow external model types to penetrate the Domain Layer.
- Be owned by the downstream (consumer) context.
- Be independently testable against both the external contract and the local domain contract.

#### 7.4 Refactoring Governance under DDD

Refactoring SHALL be evaluated against domain model correctness, not only structural metrics.

**Validation Gate — A refactor is acceptable only if ALL of the following hold:**

**S1 — Domain Integrity Validation:**
- Aggregate invariants remain fully enforced post-refactor.
- No implicit shared state introduced across Bounded Context boundaries.
- Failure and error semantics are preserved and aligned with domain expectations.
- ACL contracts remain unbroken.

**S2 — Model Clarity Validation:**
- Ubiquitous Language alignment has improved or is maintained.
- Building block classifications (Entity, VO, Aggregate, etc.) remain correct.
- Documentation and Context Map remain synchronized.
- Cognitive load for a new developer reading the model has decreased.

**S3 — Scalability Validation:**
- Bounded Context boundaries have not been widened unnecessarily.
- No new coupling introduced between previously independent Aggregates.
- Domain Event contracts remain backward-compatible or are explicitly versioned.
- No performance complexity regression.

> **Rule:** A refactor that improves structural metrics but degrades Ubiquitous Language clarity or blurs Bounded Context boundaries is **non-compliant and MUST be reverted**.

**Each refactor step MUST be:**
1. **Independently compilable** — No broken intermediate states.
2. **Domain-verifiable** — All domain invariant tests pass after the step.
3. **Reversible** — A clean rollback MUST be possible.

Refactoring MUST NOT be bundled with feature changes in the same commit.

---

### Section 8 — Quality Assurance in a DDD Context

Quality assurance in a DDD system is structured around **domain model correctness** as the primary quality signal, with tests organized to mirror the domain's own structure.

#### 8.1 Test Taxonomy

| Test Type | Scope | Validates |
|-----------|-------|-----------|
| **Domain Unit Test** | Single Entity, Value Object, or Aggregate | Business rules, invariants, and state transitions using only domain objects |
| **Domain Service Test** | Domain Service in isolation | Policy correctness using domain object stubs |
| **Application Service Test** | Use-case orchestration | Correct sequencing of domain operations; uses mocked Repositories and Domain Services |
| **ACL Contract Test** | Anti-Corruption Layer | Correct translation of external model ↔ local domain model |
| **Context Integration Test** | Cross-context boundary | Correct message/event flow across Bounded Context boundaries |
| **Repository Adapter Test** | Infrastructure adapter | Correct persistence and retrieval of Aggregates |

#### 8.2 Domain-Centric Test Naming

Tests SHALL be named using the Ubiquitous Language of the Bounded Context, not technical implementation terms.

*Recommended structure:* `[AggregateOrService]_[domainScenario]_[expectedDomainOutcome]`

*Examples:*
- `Order_whenItemAddedExceedingStockLimit_raisesInventoryExceededEvent`
- `PricingPolicy_whenApplyingVIPDiscountToEligibleOrder_returnsDiscountedTotal`
- `PaymentACL_whenReceivingStripeWebhookEvent_translatesIntoPaymentReceivedDomainEvent`

#### 8.3 Invariant Verification Standard

Every Aggregate Root MUST have tests that explicitly verify each of its declared invariants.

- Invariants SHALL be documented as part of the Aggregate's specification before implementation.
- Tests for invariant violations MUST verify that the Aggregate raises an explicit domain exception (not a generic runtime error).
- No invariant test MAY mock internal Aggregate components; the full Aggregate is the unit under test.

#### 8.4 Coverage and Regression Policy

| Metric | Rule |
|--------|------|
| Domain layer coverage | SHALL NOT decrease; targeting ≥90% line coverage on domain objects |
| Aggregate invariant coverage | 100% — every declared invariant MUST have a corresponding test |
| ACL translation coverage | 100% — every translation path MUST be tested in both directions |
| Integration test regression | Zero tolerance — all context boundary tests MUST pass before merge |
| Performance complexity | No time or space complexity regression permitted |

---

## PART IV — AI AGENT BEHAVIORAL CONSTRAINTS

---

### Section 8 — Zero-Hallucination Mandate (Determinism Control — S1)

This section defines the behavioral bounds that constrain any AI agent operating within this system to prevent unpredictable, invented, or unverifiable outputs.

#### 8.1 Prohibition on Invention

The agent SHALL NOT invent, assume, fabricate, or mock any of the following unless **explicitly instructed under a designated test isolation context with documented boundary stubs**:

- Internal API signatures
- Database schemas or table structures
- External service contracts or response formats
- Library or package behaviors not evidenced in provided context
- Business rules not stated in the requirements

> **Violation Classification:** Inventing undocumented system elements is a **Secure (S1) violation** and MUST cause the agent to halt and request clarification.

#### 8.2 Fail-Fast Protocol

The agent SHALL prioritize **early, explicit failure over plausible-but-unverified output.**

Trigger conditions for a halt-and-clarify response:

| Condition | Required Action |
|-----------|----------------|
| Ambiguous user intent | Request clarification BEFORE generating code or architecture |
| Missing technical context | Explicitly list what is missing and halt |
| Contradictory requirements | Surface the contradiction and request resolution |
| Undocumented dependency required | Flag and request documentation |

**The agent MUST NOT resolve ambiguity through assumption.** Contextual guessing is a S1 violation.

#### 8.3 Output Strictness and Bounded Generation

All agent outputs SHALL be:

- **Structured** — Code, diffs, architectural diagrams, or clearly labeled explanations.
- **Scoped** — Strictly addressing the requested engineering artifact.
- **Verifiable** — Every claim or code snippet must be traceable to a requirement, a documented API, or an explicit instruction.

**Prohibited output patterns:**

| Prohibited Pattern | Reason |
|-------------------|--------|
| Unsolicited architectural opinions not grounded in 3S | Out of scope |
| Philosophical justifications outside this standard | Not verifiable |
| Conversational filler or hedging language | Reduces signal clarity |
| Speculative "this might be useful" additions | Violates minimal-implementation rule |

#### 8.4 Human-in-the-Loop (HITL) Enforcement

The agent SHALL NOT autonomously execute or advise any of the following without **explicit human authorization**:

- File deletion or directory removal
- Database schema drops or destructive migrations
- Force-push to version control
- Permission escalation
- Production environment modifications

All proposed structural changes MUST be presented as **dry-runs or annotated diffs** and explicitly confirmed before integration.

---

### Section 9 — Interaction and Communication Protocol

#### 9.1 Clarification Before Action

When a request is ambiguous, incomplete, or contradictory, the agent MUST:

1. **Stop** — Do not proceed with any code generation or structural modification.
2. **Enumerate** — List the specific ambiguities or missing information.
3. **Ask** — Pose a concise, targeted clarifying question (one issue per question where possible).
4. **Wait** — Await a response before resuming.

#### 9.2 Structured Response Format

Agent responses SHALL follow this structure when delivering engineering output:

```
[CONTEXT ASSESSMENT]
  - Understood requirements
  - Identified constraints
  - Flagged ambiguities (if any)
  - Identified Bounded Context(s) affected

[PROPOSED APPROACH]
  - DDD building blocks involved (Aggregate, Entity, Value Object, Domain Event, etc.)
  - Context Map impact: any new or modified cross-context relationships
  - Ubiquitous Language terms introduced or modified
  - 3S impact assessment

[IMPLEMENTATION]
  - Code / diff / architectural artifact
  - Domain model changes clearly labeled by building block type

[VALIDATION SUMMARY]
  - Which domain invariants are satisfied or introduced
  - Which Bounded Context boundaries are respected or modified
  - Which 3S dimensions are improved
  - Remaining risks or open questions
```

#### 9.3 Minimal Footprint Principle

The agent SHALL make the smallest change necessary to satisfy the requirement. Over-engineering, gold-plating, or adding unrequested features is prohibited.

---

## PART V — LIFECYCLE AND PROCESS GOVERNANCE

---

### Section 10 — Lifecycle Execution Model (ISO/IEC 12207 Aligned)

All engineering activities SHALL follow this controlled execution workflow:

```
STEP 1 — DOMAIN DISCOVERY AND REQUIREMENT COMPREHENSION
  Identify the subdomain(s) affected: Core, Supporting, or Generic.
  Extract and validate Ubiquitous Language terms with domain stakeholders.
  Identify acceptance criteria expressed in domain language.
  Surface ambiguities → Resolve via Section 9.1 before proceeding.

STEP 2 — BOUNDED CONTEXT AND MODEL SCOPING
  Identify which Bounded Context(s) are affected.
  Determine if a new Bounded Context is required or if an existing one expands.
  Assess Context Map impact: any new or modified inter-context relationships.
  Classify all domain concepts involved using DDD building blocks (Section 6.3).

STEP 3 — DOMAIN MODEL DESIGN
  Define or update Aggregate boundaries and Aggregate Roots.
  Specify all invariants the Aggregate Root must enforce.
  Define Domain Events raised as side effects of state changes.
  Identify required Repositories (interfaces only; no infrastructure details).
  Design ACL translation contracts for any external integration (Section 7.3).

STEP 4 — ARCHITECTURAL IMPACT ASSESSMENT
  Evaluate SOC implications (Section 1).
  Assess dependency changes (Section 2).
  Identify boundary contract modifications (Section 3).

STEP 5 — EXPLICIT HUMAN APPROVAL
  Present the domain model design, Context Map delta, and decomposed plan.
  Await confirmation before any implementation begins.
  Unapproved structural modification or Bounded Context boundary change is prohibited.

STEP 6 — IMPLEMENTATION (DDD-Modular)
  Implement Domain Layer first: Entities, Value Objects, Aggregates, Domain Events.
  Implement Domain Services and Application Services next.
  Implement Infrastructure adapters (Repositories, ACLs) last.
  Each implementation unit SHALL have corresponding domain invariant tests (Section 8).

STEP 7 — VALIDATION
  Verify all domain invariant tests pass.
  Verify ACL translation tests pass in both directions.
  Verify no coverage regression.
  Verify no complexity regression.
  Verify documentation and Context Map parity.

STEP 8 — FORMAL REPORTING
  Deliver a consolidated summary per Section 9.2.
  Commit with accurate, descriptive diff representation.
  Update Context Map document if inter-context relationships changed.
```

---

### Section 11 — Documentation Integrity

Documentation is treated as a **first-class engineering artifact**, not a secondary deliverable.

#### 11.1 Sync or Sink Principle

Code and documentation MUST remain synchronized at all times. A discrepancy between implementation and documentation is classified as a **governance failure**.

#### 11.2 Documentation Standards

| Requirement | Rule |
|-------------|------|
| Technical depth | SHALL NOT be reduced during refactoring or updates |
| Architectural rationale | MUST remain explicit and current |
| Public API documentation | MUST include descriptions, parameter semantics, return values, and behavioral examples |
| Inline comments | Reserved for non-obvious logic; obvious code SHALL NOT be over-commented |

#### 11.3 Automated API Documentation

Where the language supports it, documentation SHALL be generated from structured docstrings/annotations (e.g., JSDoc, Rustdoc, Javadoc, Python docstrings) to enforce parity between code and documentation.

---

### Section 12 — Security and Data Protection

#### 12.1 Data Minimization

The system SHALL NOT store, request, infer, or record personal or sensitive data beyond what is strictly required by the stated functional requirement.

#### 12.2 Attribute Sovereignty and Immutability

Critical system-level attributes SHALL be treated as immutable unless a specific, documented business requirement justifies modification:

| Attribute Class | Default Policy |
|----------------|---------------|
| System-generated IDs | Immutable — no UI or API write access |
| Audit-critical fields (timestamps, actor IDs) | Immutable — system-controlled only |
| Privilege escalation flags (e.g., superadmin) | Immutable — no user-editable interface |
| Cryptographic keys and secrets | Write-once — rotation via dedicated key management only |

Providing UI or API write access to these attributes requires **explicit business justification and security review**.

#### 12.3 Security Overrides Flexibility

Security constraints override architectural flexibility. An architectural pattern that introduces a security weakness is non-compliant regardless of its other merits.

---

## PART VI — ENGINEERING BASELINE AND OPERATIONAL SAFEGUARDS

---

### Section 13 — Engineering Baseline Control

A valid engineering state is defined by three synchronized baselines:

| Baseline | Contents | Verification Method |
|----------|----------|---------------------|
| **Architecture Baseline** | Layer structure, Bounded Context map, dependency graph, boundary contracts, ADRs | Automated architecture tests, dependency analysis, Context Map review |
| **Domain Model Baseline** | Aggregate definitions, invariant specifications, Ubiquitous Language glossary, Domain Event catalog | Domain invariant test suite; UL consistency audit |
| **Requirements Baseline** | Authoritative specifications, acceptance criteria expressed in UL | Traceability matrix |
| **Codebase Baseline** | Stable, verified implementation state | Continuous passing domain invariant and ACL test suite |

**Deviation from any baseline requires explicit justification documented in an Architecture Decision Record (ADR).**

---

### Section 14 — Operational Safeguards

These are unconditional operational constraints that apply to all agents at all times:

#### 14.1 Scanning and File Operation Safety

- Agents SHALL apply strict directory filtering when scanning codebases (exclude `node_modules/`, `vendor/`, `.git/`, `target/`, `dist/`, `build/`, and other generated artifact directories).
- Unfiltered directory scanning is **prohibited** for reasons of efficiency and system integrity.

#### 14.2 Destructive Operation Protocol

| Operation | Required Protocol |
|-----------|-------------------|
| File deletion | Dry-run → human confirmation → execute |
| Force push | Explicit justification → human authorization → execute |
| Database drop/truncation | Impact assessment → human authorization → execute |
| Overwriting complex/large files | Prefer surgical targeted edits over full overwrite |

**"Push" explicitly means: stage, commit with a descriptive message, and push.** It does not mean force-push unless explicitly stated.

#### 14.3 Language-Specific Quality Requirements

When operating in strongly-typed compiled languages (e.g., Rust, Go, Kotlin), the following additional requirements apply:

- All public items MUST include comprehensive inline documentation.
- Examples MUST be provided in documentation for non-trivial public functions.
- Macro-based abstractions (e.g., `macro_rules!` in Rust) SHOULD be used when refactoring code with repetitive structural patterns, to enforce DRY compliance.
- All outputs MUST compile cleanly with zero warnings under the strictest applicable linter/compiler settings.

---

## APPENDICES

---

### Appendix A — Violation Classification Reference

| Violation | Classification | Required Response |
|-----------|---------------|-------------------|
| Domain layer imports infrastructure | SOC violation (S1) | Halt, refactor, retest |
| Agent invents undocumented API or domain rule | Hallucination (S1) | Halt, request documentation |
| Duplicated business logic across Bounded Contexts | DRY + DDD violation (S3) | Refactor to SSOT within correct BC |
| Aggregate invariant enforced outside the Aggregate Root | DDD structural violation (S1) | Refactor; move enforcement to AR |
| External type leaks past the ACL into the Domain Layer | ACL breach (S1) | Halt, implement translation in ACL |
| Ubiquitous Language term used inconsistently across a BC | UL violation (S2) | Audit and align all usages |
| Domain Event named in present or future tense | DDD naming violation (S2) | Rename to past-tense domain action |
| Cross-Aggregate direct object reference (not by ID) | Aggregate boundary violation (S1) | Replace with identity reference |
| Code merged without passing domain invariant tests | Quality failure (S1) | Block merge, restore passing state |
| Documentation or Context Map diverged from code | Governance failure (S2) | Synchronize before next commit |
| Hardcoded configuration value in domain logic | Hygiene violation (S3) | Extract to configuration or Value Object |
| Structural change without approval | Process violation (S1) | Revert, seek approval |
| Ambiguity resolved by assumption | Hallucination (S1) | Revert, request clarification |
| New Bounded Context introduced without Context Map update | Architecture governance failure (S2) | Update Context Map before proceeding |

---

### Appendix B — Architecture Decision Record (ADR) Template

When a deviation from this standard is required, it MUST be documented using the following structure:

```
ADR-[NUMBER]: [Short Title]

Date: [YYYY-MM-DD]
Status: [Proposed | Accepted | Deprecated | Superseded]
Deciders: [Names or roles]

Context:
  [Describe the situation and forces at play.]

Decision:
  [Describe the change being proposed or accepted.]

Deviation from Standard:
  [Which section of this document is being deviated from, and why.]

3S Impact Assessment:
  S1 (Secure):  [Impact and mitigation]
  S2 (Sustain): [Impact and mitigation]
  S3 (Scalable):[Impact and mitigation]

Consequences:
  [What becomes easier or harder as a result?]

Review Date:
  [When this decision should be revisited.]
```

---

### Appendix C — Behavioral Decision Tree for Agents

Use this tree to determine the correct action for any given request:

```
START
  │
  ├─ Is the request clear and unambiguous?
  │    ├─ NO → Section 9.1: Halt. Enumerate ambiguities. Ask. Wait.
  │    └─ YES ↓
  │
  ├─ Does fulfilling the request require undocumented context
  │  (APIs, schemas, domain rules, Ubiquitous Language terms)?
  │    ├─ YES → Section 8.1: Halt. List missing information. Request docs.
  │    └─ NO ↓
  │
  ├─ Does the request involve a destructive operation?
  │    ├─ YES → Section 8.4: Present dry-run. Await explicit authorization.
  │    └─ NO ↓
  │
  ├─ Have the affected Bounded Context(s) been identified?
  │    ├─ NO → Section 6.1: Identify and name Bounded Contexts first.
  │    └─ YES ↓
  │
  ├─ Are all domain concepts classified into DDD building blocks?
  │  (Entity, Value Object, Aggregate, Domain Service, Domain Event)
  │    ├─ NO → Section 6.3: Classify before generating code.
  │    └─ YES ↓
  │
  ├─ Does the change cross a Bounded Context boundary?
  │    ├─ YES → Section 7: Select and document the integration pattern.
  │    │         Does it require an ACL?
  │    │           ├─ YES → Section 7.3: Implement ACL. Test both directions.
  │    │           └─ NO → Document the chosen pattern in Context Map. ↓
  │    └─ NO ↓
  │
  ├─ Are all Aggregate invariants explicitly defined and tested?
  │    ├─ NO → Section 8.3: Define invariants. Write invariant tests first.
  │    └─ YES ↓
  │
  ├─ Have all domain invariant and ACL tests passed?
  │    ├─ NO → Section 8.4: Halt. Restore passing state. Investigate.
  │    └─ YES ↓
  │
  ├─ Does the change improve at least one 3S dimension
  │  without degrading UL clarity or BC boundaries?
  │    ├─ NO → Section 7.4: Refactor is non-compliant. Revert or redesign.
  │    └─ YES ↓
  │
  └─ DELIVER: Structured output per Section 9.2. Update Context Map if needed.
             Commit with accurate, descriptive diff.
```

---

### Appendix D — Glossary

| Term | Definition |
|------|------------|
| **ACL** | Anti-Corruption Layer — a translation boundary that protects a domain model from contamination by external or foreign models. |
| **ADR** | Architecture Decision Record — a document capturing an architectural decision and its context. |
| **Aggregate** | A cluster of Entities and Value Objects treated as a single transactional unit, accessed exclusively through its Aggregate Root. |
| **Aggregate Root (AR)** | The single entry point to an Aggregate; responsible for enforcing all invariants within the Aggregate boundary. |
| **Bounded Context (BC)** | An explicit logical boundary within which a specific domain model and Ubiquitous Language are valid and consistently applied. |
| **Boundary Contract** | An explicit, typed interface definition at an architectural boundary. |
| **Context Map** | A document that describes all Bounded Contexts in a system and the integration patterns governing the relationships between them. |
| **DDD** | Domain-Driven Design — an approach to software development that centers the design on the core domain and domain logic. |
| **Domain Event** | An immutable record of a significant occurrence in the domain, named in the past tense (e.g., `OrderPlaced`). |
| **Domain Service** | A stateless service encapsulating domain logic that does not naturally belong to any single Entity or Value Object. |
| **DRY** | Don't Repeat Yourself — every knowledge element has a single, authoritative source. |
| **Entity** | A domain object with a persistent, unique identity that distinguishes it from other objects regardless of attribute values. |
| **HITL** | Human-in-the-Loop — a required human authorization step before executing high-impact operations. |
| **Repository** | An abstraction (interface) for persisting and retrieving Aggregates, defined in the Domain Layer, implemented in the Infrastructure Layer. |
| **SOC** | Separation of Concerns — the architectural principle of isolating distinct responsibilities. |
| **SSOT** | Single Source of Truth — one authoritative location for a given piece of knowledge or logic. |
| **Ubiquitous Language (UL)** | The shared, unambiguous vocabulary derived from domain expertise, used uniformly in all code, documentation, and communication within a Bounded Context. |
| **Value Object (VO)** | An immutable domain object defined entirely by its attribute values, with no persistent identity. |
| **3S** | The governing doctrine: Secure, Sustain, Scalable. |
| **Zero-Hallucination** | The mandate that agents MUST NOT invent, assume, or fabricate system artifacts, domain rules, or API contracts. |

---
