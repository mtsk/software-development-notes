(In-Progress) Vaughn Vernon: Implementing Domain-Driven Design

---

## DDD organizational and integration patterns

**Partnership**
→ Teams work as equals, jointly designing and evolving closely coupled models.  
**Shared Kernel**
→ Teams share and jointly maintain a small, common part of the domain model.  
**Customer-Supplier Development**
→ One team owns a system or model while another depends on it, influencing changes through clear requirements and ongoing feedback.  
**Conformist**
→ One team fully adopts another team’s model and interfaces as they are, making no attempt to influence or change them.  
**Anticorruption Layer**
→ A dedicated translation layer isolates a system from an external model by converting between internal concepts and outside interfaces.  
**Open Host Service**
→ A system exposes a well-defined, stable interface that multiple external consumers can use consistently.  
**Published Language**
→ A shared, well-documented language defines how systems exchange information, ensuring consistent meaning across boundaries.  
**Separate Ways**
→ Two systems operate independently, with no integration or shared model between them.  
**Big Ball of Mud**
→ A system lacks clear structure, with tangled, unorganized code and dependencies throughout.

## Architecture
### Layered Architecture
Layered Architecture structures a system into horizontal layers such as **User Interface**, **Application**, **Domain**, and **Infrastructure**, each with clear responsibilities. The domain layer contains business logic and rules, while outer layers depend inward but not vice versa. This structure makes systems easier to understand and maintain, but it can degrade into an anemic domain model if business logic leaks into application services or infrastructure.

**Best suited for:** Traditional enterprise systems with moderate complexity and strong transactional requirements.  
**Key risk:** Over-coupling between layers and procedural domain logic.

### Ports and Adapters (Hexagonal Architecture)
Ports and Adapters places the **domain model at the center** and defines explicit interfaces (ports) for all external interactions. Infrastructure concerns such as databases, UIs, messaging systems, and APIs are implemented as interchangeable adapters. This approach emphasizes **testability, isolation, and long-term maintainability**, allowing the domain model to evolve independently of technical details. Frequently used as an overarching style.

**Best suited for:** Systems with a rich domain model and long lifespan.  
**Key risk:** Higher upfront design effort and conceptual complexity.

### Service-Oriented Architecture (SOA)
Service-Oriented Architecture decomposes the system into **coarse-grained services** that communicate via well-defined contracts, often using messaging or remote procedure calls. Each service encapsulates its own business capabilities and may align with bounded contexts. SOA focuses on **integration, reuse, and organizational scalability**, but it can suffer from excessive coupling if services share data models or become chatty.

**Best suited for:** Large enterprises integrating multiple systems and teams.  
**Key risk:** Distributed monoliths and governance overhead.

### Representational State Transfer (REST)
REST is an architectural style for distributed systems that models domain concepts as **resources** accessed via standard HTTP verbs (GET, POST, PUT, PATCH, DELETE, ..). It enforces stateless communication and uniform interfaces, improving scalability and interoperability. In a DDD context, REST works best when resource boundaries align with aggregates, but it can be limiting when domain behavior does not map cleanly to CRUD semantics.

**Best suited for:** Public APIs and client-server integrations.  
**Key risk:** Oversimplifying domain behavior into CRUD operations.

### Command–Query Responsibility Segregation (CQRS)
CQRS separates **commands** (state-changing operations) from **queries** (read-only access), often using different models and storage strategies for each. This separation allows each side to be optimized independently and supports complex domain behavior and scalability. CQRS is frequently paired with event-driven systems but introduces additional complexity in synchronization and consistency.

**Best suited for:** Complex domains with high read/write asymmetry.  
**Key risk:** Increased architectural and operational complexity.

### Event-Driven Architecture
Event-Driven Architecture uses **domain events** to communicate state changes asynchronously between components or bounded contexts. Producers emit events without knowing who consumes them, enabling loose coupling and scalability. This style aligns naturally with DDD by making domain events explicit, but it requires careful handling of eventual consistency and failure scenarios.

**Best suited for:** Highly decoupled, reactive, or distributed systems.  
**Key risk:** Debugging difficulty and eventual consistency challenges.

### Data Fabric / Big Data Architecture
Data Fabric architecture integrates multiple data sources, processing pipelines, and analytical models into a unified data platform. It supports large-scale data ingestion, transformation, and analysis across the organization. In DDD, this style is typically **adjacent** to core domain systems, serving reporting, analytics, or machine-learning use cases rather than transactional domains.

**Best suited for:** Analytics-heavy and data-driven systems.  
**Key risk:** Mixing analytical concerns with core domain logic.

## Entities
An **Entity** is an object defined by its **identity**, not its attributes. Its data may change over time, but its identity remains constant. An Entity models something that must be uniquely identifiable and consistent throughout its lifetime.

* Have a lifecycle and continuity.
* Encapsulate business behavior, not just data.
* Protect invariants and prevent invalid state changes.
* Compare equality based only on identity.
* Typically live inside Aggregates and are accessed through the Aggregate Root.

### Identity Generation
* Database-generated
* Application-generated -> prefered when possible, as they allow full object creation before persistence and reduce coupling to infrastructure
* UUID-based

## Value Objects
A **Value Object** is defined by its **attributes**, not identity. Two value objects with the same values are considered equal. Favor the use of Value Objects whenever possible, because they are easier to develop, test and maintain.

* Are **immutable**.
* Have **no conceptual identity**.
* Are replaced, not modified, when something changes.
* Encapsulate domain concepts and related logic.
* Help make the model more expressive and precise.

-> Design small, side-effect-free Value Objects to model important domain concepts (e.g., Money, Address, DateRange) instead of using primitive types.

## Domain Services

A Domain Service models significant business operations that don’t naturally fit within an Entity or Value Object, keeping the domain model expressive without becoming anemic.

Services are used when:
* A business operation involves multiple aggregates.
* The behavior doesn’t conceptually fit inside a single object.
* The operation represents an important domain concept.

Key characteristics:
* **Stateless** (no internal mutable state).
* Express domain intent clearly.
* Operate on domain objects passed as parameters.
* Contain business rules — not infrastructure logic.

-> Be carefull about creating “service-heavy” models that push behavior out of Entities. Services should be used sparingly — only when behavior truly doesn’t belong elsewhere.

## Domain Events

A **Domain Event** represents something important that **happened in the domain** (past tense), such as *OrderPlaced* or *PaymentReceived*.

Domain Events:
* Capture **meaningful business occurrences**.
* Are **immutable** once created.
* Help **decouple** parts of the system.
* Enable communication between aggregates or bounded contexts.
* Can trigger side effects (e.g., notifications, integrations).

They should be:
* Named in **business language**.
* Focused on what happened, not how.
* Published after state changes are committed.

Key points:
* **Value vs. Aggregate Events**
  * **Value-based (common):** simple, immutable fact with no lifecycle
  * **Aggregate-like (rare):** used when the event must be tracked, queried, or managed over time
* **Publishers & Subscribers**
  * **Publishers:** usually **Aggregates** (sometimes Application Services)
  * **Subscribers:** handlers in Application/Domain Services or external systems
* **De-duplication**
  * Events may be delivered multiple times → subscribers must be **idempotent** to avoid duplicate side effects
* **Event Store**
  * Append-only storage of events (DB, message store, or specialized tool)
  * Enables **audit history** and **event sourcing** (rebuilding state)
 
## Modules
 
Modules structure the domain model into meaningful, loosely coupled groups, making complex systems easier to understand and evolve.

* Reflect **business concepts**, not technical layers
* Promote **high cohesion** within a module and **low coupling** between modules
* Provide clear **boundaries and naming** to improve understanding
* Help manage complexity by structuring large models
* Maps to 'namespace' in C# or 'package' in Java
* When the boundary between Module and Bounded Context is fuzzy, prefer Module
* Name Modules per Ubiquitous Language

Example - Identity & Access Context:
```
identityaccess
├── domain
│   ├── model
│   │   ├── identity
│   │   │   ├── Tenant
│   │   │   ├── User
│   │   │   ├── Group
│   │   │   └── Role
│   │   │
│   │   ├── access
│   │   │   ├── Permission
│   │   │   └── RoleAssignment
│   │   │
│   │   └── common
│   │       ├── DomainEvent
│   │       └── Identifier
│   │
│   ├── service
│   │   └── AuthenticationService
│   │
│   └── event
│       ├── UserRegistered
│       └── TenantProvisioned
```

## Aggregates
An Aggregate defines a transactional consistency boundary around related domain objects, with an Aggregate Root enforcing business rules (invariants) and protecting the integrity of the model.

At the center is the **Aggregate Root**:
* The only object accessible from outside the Aggregate
* Responsible for enforcing consistency rules
* Controls access to internal Entities and Value Objects

### Key Principles
* Keep Aggregates **small and focused** - large Aggregates cause contention, performance problems, complex transactions, ..
* Modify only **one Aggregate per transaction** when possible
* Reference other Aggregates **by identity**, not direct object references
* Design around **true business invariants**, not object navigation convenience
* Use **Domain Events** for cross-Aggregate coordination

### Consistency
* Inside an Aggregate → **strong consistency**
* Between Aggregates → usually **eventual consistency**

### Aggregate Design Example
```text
Order (Aggregate Root)
├── OrderItem
├── ShippingAddress
└── PaymentInformation
```
External code interacts only with `Order`, not directly with `OrderItem`.

## Factories
A **Factory** encapsulates the creation of complex domain objects or Aggregates, ensuring they are created in a **valid and consistent state**.

Factories are useful when:
* Object creation is **complex**
* Multiple objects must be assembled together
* Creation logic would clutter Entities or Application Services
* Invariants must be enforced during construction

### Key Principles
* Factories create **fully valid Aggregates**
* They hide complicated construction details
* They should express **domain intent**
* Simple objects usually do **not** need factories

Factories may be implemented as:
* Dedicated Factory classes
* Static factory methods
* Aggregate creation methods

### Example
```text id="c1j4x7"
TenantProvisioningFactory
 ├── creates Tenant
 ├── creates Administrator User
 └── assigns initial Role
```

The caller requests a new Tenant, while the Factory handles all required setup consistently.

## Repositories
A Repository provides the illusion of an **in-memory collection of Aggregates**, hiding persistence details from the domain model. It provides access to Aggregate Roots while hiding persistence details, allowing the domain model to focus on business logic and consistency rather than storage concerns.

### Key Principles
* A Repository is defined **per Aggregate Root**, not per Entity.
* It provides simple operations such as:
  * `add()`
  * `remove()`
  * `findById()`
  * selected business-oriented queries
* The **domain depends only on the Repository interface**; the implementation belongs to the infrastructure layer.
* Repositories should return **fully initialized Aggregates**.

### Repository Design Guidelines
* Create **one Repository per Aggregate Root**.
* Retrieve Aggregates primarily by their **identity**.
* Avoid generic CRUD repositories with numerous arbitrary queries.
* Avoid exposing persistence concepts (tables, joins, ORM details) to the domain.
* Use **specialized query services** for complex reporting and read-only views instead of forcing everything through Repositories.

### Collection-Oriented vs. Persistence-Oriented Repositories
| Collection-Oriented (Preferred by Vernon)    | Persistence-Oriented                     |
| -------------------------------------------- | ---------------------------------------- |
| Models an in-memory collection of Aggregates | Closely resembles database access        |
| Domain-focused interface                     | Persistence-focused interface            |
| Small set of meaningful methods              | Often exposes many CRUD/query operations |
| Hides storage details completely             | May leak ORM or database concerns        |

### Repositories and CQRS
* **Command side** → Aggregates + Repositories for transactional consistency
* **Query side** → Optimized read models and query services, implement as Application Services
This separation keeps Repositories simple and focused on consistency.

### Implementation
Repository implementations may use:
* Relational databases with an ORM
* Document databases
* Event Stores
* Any persistence mechanism, as long as the domain remains persistence-ignorant

---
