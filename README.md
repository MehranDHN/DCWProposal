# Tehran Construction Ontology & Distributed Map-Based Platform

**Private repository** – Ontology-driven foundation for a distributed, map-based construction permitting and compliance application focused on Tehran Municipality.

## Vision

Tehran is Iran’s largest city and one of the most complex urban environments for construction. Building regulations, zoning rules, geotechnical requirements, and administrative procedures vary significantly across its **22 municipal districts (مناطق)**.  

This project treats an **OWL ontology** as the single ubiquitous language shared between:
- Business domain experts (municipality officers, architects, geotechnical engineers)
- Technical teams (developers, data engineers, GIS specialists)

The ontology starts from the **Building Topology Ontology (BOT)** and is systematically extended to cover:
- Administrative geography of Tehran
- Construction documents, plans, and design artefacts
- Materials and BIM data (especially from Autodesk Revit / IFC)
- Municipal policies and rules
- Workflows and permit processes

A rule engine (SHACL + SPARQL / external engine) evaluates applications against location-specific policies. The map interface becomes a pure visualization and interaction layer over the knowledge graph.

## Problem Statement

- Construction rules differ by district.
- Physical building topology, legal parcels, design documents, geotechnical reports, foundation designs, and material specifications must stay linked.
- Stakeholders need a shared, precise vocabulary.
- The system must support distributed data (different districts / teams can maintain their own named graphs) while remaining interoperable.
- Future national or municipal code changes should be additive, not disruptive.

## Design Principles

1. **BOT remains pure** – It only describes physical topology and containment.
2. **Modular extension** – New concerns live in separate ontology modules that reference BOT entities.
3. **Ubiquitous language first** – Classes and properties are chosen so both domain experts and developers can use the same terms.
4. **Rule-engine friendly** – Policies are first-class citizens that can be queried and evaluated.
5. **Map-native** – Every relevant entity can carry GeoSPARQL geometry.
6. **BIM-aware** – Revit / IFC models and their materials are first-class information resources linked to the topology.

## Ontology Architecture (Layered)

```
┌─────────────────────────────────────────────────────────┐
│  Process / Workflow Layer                               │
│  (PermitApplication, Workflow, Step, State, Decision)   │
├─────────────────────────────────────────────────────────┤
│  Regulatory & Policy Layer                              │
│  (MunicipalPolicy, ConstructionRule, Constraint)        │
├─────────────────────────────────────────────────────────┤
│  Design & Documentation Layer                           │
│  (Plans, Geotechnical reports, Foundation designs,      │
│   BIM models, Material specifications)                  │
├─────────────────────────────────────────────────────────┤
│  Material & Product Layer                               │
│  (Material, BuildingProduct – Revit families / IFC)     │
├─────────────────────────────────────────────────────────┤
│  Administrative & Spatial Layer                         │
│  (District, Sub-district, LandParcel, ZoningClass)      │
├─────────────────────────────────────────────────────────┤
│  Core Topology – BOT (unchanged)                        │
│  Site → Building → Storey → Space → Element             │
└─────────────────────────────────────────────────────────┘
         + GeoSPARQL geometries on spatial entities
```






### 1. TTL file (combined entry point)



### 2. Brief version already added to README

**Ontology at a Glance (Brief)**

We keep the official **Building Topology Ontology (BOT)** pure for physical structure  
(`bot:Site` → `bot:Building` → `bot:Storey` → `bot:Space` → `bot:Element`).

Two extension modules add the rest of the domain:

| Module | File | Main content |
|--------|------|--------------|
| **Core** | `ontology/mdhn-core.ttl` | 22 Administrative Districts, Land Parcels, Zoning Classes + spatial relations + GeoSPARQL readiness |
| **Documentation** | `ontology/mdhn-documentation.ttl` | Plans, Geotechnical Reports, Foundation Designs, BIM Models (Revit/IFC), Materials + typed links to BOT entities |
| **Full entry point** | `ontology/mdhn-full.ttl` | Imports both modules + BOT |

**Key linking pattern**  
Documents and materials are attached to the most specific BOT entity possible:

- Site / Parcel → Geotechnical report, Site plan, Zoning map  
- Building → Foundation design, Architectural & Structural plans, BIM model  
- Element → Material / Product type  

This keeps the rule engine simple (“does this building in District 5 have an approved FoundationDesign?”) and the map UI rich (click → topology + all attached documents).

---

### 1. Core – Building Topology Ontology (BOT)

Namespace: `https://w3id.org/bot#`  
Prefix: `bot:`

We keep BOT completely intact. Key classes used:
- `bot:Zone` (superclass)
- `bot:Site`
- `bot:Building`
- `bot:Storey`
- `bot:Space`
- `bot:Element`
- `bot:Interface`

BOT already provides `bot:has3DModel` and `bot:hasSimple3DModel` – useful hooks for geometry coming from Revit/IFC.

### 2. Administrative & Spatial Extensions

New classes (namespace `mdhn:`):
- `mdhn:AdministrativeDistrict` – the 22 مناطق of Tehran
- `mdhn:SubDistrict` / `mdhn:Neighborhood`
- `mdhn:LandParcel` – cadastral unit
- `mdhn:ZoningClass`

Key relations:
- `mdhn:locatedInDistrict`
- `mdhn:subjectToZoning`
- `bot:containsZone` refined with district-level containment
- GeoSPARQL (`geo:hasGeometry`, `geo:asWKT`) on sites, parcels, and districts

### 3. Design & Documentation Layer

This is the major extension requested for maps, plans, geotechnical documents, foundation designs, and BIM artefacts.

**Classes**
- `mdhn:InformationResource` (superclass)
  - `mdhn:DesignDocument`
    - `mdhn:ArchitecturalPlan`
    - `mdhn:StructuralPlan`
    - `mdhn:FoundationDesign`
    - `mdhn:GeotechnicalReport`
    - `mdhn:SitePlan` / `mdhn:ZoningMap` / `mdhn:CadastralMap`
  - `mdhn:BIMModel` (Revit project, IFC file, etc.)
  - `mdhn:MaterialSpecification`

**Properties** (can attach to any `bot:Zone` or `bot:Element`)
- Generic: `mdhn:hasInformationResource` / `mdhn:documents`
- Typed (preferred for rules and UI):
  - `mdhn:hasFoundationDesign`
  - `mdhn:hasGeotechnicalReport`
  - `mdhn:hasArchitecturalPlan`
  - `mdhn:hasStructuralPlan`
  - `mdhn:hasSitePlan`
  - `mdhn:hasBIMModel`

Document metadata:
- File / IIIF / download URI
- `dcterms:format`, `dcterms:created`, `dcterms:creator`
- Provenance (`prov:wasGeneratedBy` – software, author, approval status)
- Versioning (`mdhn:version`, `mdhn:supersedes`)

**Typical attachment points**

| BOT entity              | Typical attached resources                                      |
|-------------------------|-----------------------------------------------------------------|
| `bot:Site` / Parcel     | Geotechnical reports, cadastral & zoning maps, site plans       |
| `bot:Building`          | Foundation design, architectural & structural plans, main BIM model |
| `bot:Storey` / `bot:Space` | Floor plans, detailed drawings                               |
| `bot:Element`           | Material specifications, detailed Revit element data            |

### 4. Material & Product Layer

- `mdhn:Material`
- `mdhn:BuildingProduct` (Revit family/type, IFC type, manufacturer product)

Relations:
- `mdhn:hasMaterial` (domain usually `bot:Element`)
- `mdhn:hasProductType`

Properties can include Iranian standard codes, strength class, density, fire rating, etc.

**Revit / BIM integration strategy**
1. The Revit (or IFC) file itself is recorded as an `mdhn:BIMModel` linked to the building or site.
2. An import pipeline materialises the topology into BOT + material/product instances.
3. Geometry can additionally be linked via BOT’s own `bot:has3DModel`.

### 5. Regulatory & Policy Layer

- `mdhn:MunicipalPolicy`
- `mdhn:ConstructionRule`
- `mdhn:Constraint` / `mdhn:Requirement`

Important properties:
- `mdhn:appliesToDistrict`
- `mdhn:appliesToBuildingType`
- `mdhn:appliesToHeightRange` / floor-area range
- `mdhn:effectiveFrom` / `mdhn:supersededBy`
- Link to official circular / article number

This turns the “22 different rule sets” into queryable, versionable RDF.

### 6. Process / Workflow Layer

- `mdhn:PermitApplication`
- `mdhn:Workflow`
- `mdhn:WorkflowStep` / `mdhn:DecisionPoint`
- `mdhn:State` (Submitted, UnderReview, NeedsRevision, Approved, Rejected, Suspended…)

Links back to spatial and regulatory layers (`mdhn:concernsParcel`, `mdhn:evaluatedAgainstRule`).

### 7. Actor Layer

Simple role model: Applicant, LicensedEngineer, DistrictOfficer, CentralMunicipalityReviewer, Inspector, etc.

## Rule Engine Strategy

OWL alone is insufficient for production rules. Recommended hybrid:

- **SHACL** for structural constraints and cardinality.
- **SPARQL** (or SWRL for lighter cases) for policy logic.
- External rule engine (Drools, or a lightweight service) that:
  1. Receives the application + location + building parameters as a knowledge-graph fragment.
  2. Retrieves all applicable `mdhn:ConstructionRule` instances for that district.
  3. Evaluates them and writes results back as RDF (`mdhn:violatesRule`, `mdhn:satisfiesRequirement`, `mdhn:requiresDocument`).

Because documents and materials are explicitly linked, rules can require the existence and validity of specific artefacts (e.g. “District 5 + height > X → current FoundationDesign + GeotechnicalReport must exist”).

## Map Integration

- Every `bot:Site`, `mdhn:LandParcel`, and `mdhn:AdministrativeDistrict` carries GeoSPARQL geometry.
- The frontend map (Leaflet / OpenLayers / Mapbox / custom) is a pure consumer of the spatial topology + attached documents.
- Clicking a parcel or building immediately surfaces district, applicable zoning, current rules, and all linked plans / reports / materials.

## Repository Structure (proposed)

```
├── README.md                          ← this file
├── ontology/
│   ├── bot/                           ← local copy or reference to official BOT
│   ├── mdhn-core.ttl                  ← administrative + spatial extensions
│   ├── mdhn-documentation.ttl         ← plans, geotech, foundation, BIM models
│   ├── mdhn-materials.ttl             ← materials & products
│   ├── mdhn-policy.ttl                ← rules & policies
│   ├── mdhn-workflow.ttl              ← processes & states
│   └── mdhn-full.ttl                  ← imports everything
├── examples/
│   ├── sample-building.ttl
│   ├── sample-permit-application.ttl
│   └── queries/
├── shapes/                            ← SHACL shapes
├── docs/
│   ├── ontology-overview.md
│   ├── competency-questions.md
│   └── alignment-notes.md
└── scripts/
    └── (import pipelines, Revit/IFC converters, etc.)
```

## Current Status & Next Steps

**Done (conceptual design)**
- Decision to base topology on BOT
- Layered architecture defined
- Documentation & material extension pattern agreed
- Attachment points for maps, plans, geotechnical reports, foundation designs, and Revit data specified
- Rule-engine and map strategy outlined

**Immediate next actions**
1. Formalise the first Turtle modules (`mdhn-core` + `mdhn-documentation`).
2. Write a small set of competency questions and corresponding SPARQL/SHACL examples.
3. Model 1–2 real Tehran district differences as sample rules.
4. Define the minimal set of mandatory document types for the main permit workflows.
5. Sketch the import pipeline from Revit / IFC into BOT + mdhn modules.

## Namespace

```turtle
@prefix bot:  <https://w3id.org/bot#> .
@prefix mdhn: <https://w3id.org/mdhn/tehran-construction#> .   # provisional – adjust as needed
@prefix geo:  <http://www.opengis.net/ont/geosparql#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix prov: <http://www.w3.org/ns/prov#> .
```
---


## Simple flow in an Ontology-based approach

```mermaid
flowchart TD
    A[User selects parcel<br/>on Map] --> B[Resolve Polygon → LandParcel<br/>via RegistrationPlateID<br/>پلاک ثبتی]
    B --> C{Authenticate User}
    C -->|Fail| Z[Access Denied]
    C -->|Success| D[Processing Node<br/>queries core data]
    D --> E[Publish Event to Orchestrator<br/>keyed by RegistrationPlateID]
    
    E --> F1[Ownership Service]
    E --> F2[Authority Service]
    E --> F3[Geometry / Area Service]
    E --> F4[Zoning & District Service]
    E --> F5[Permit / Status Service]
    
    F1 --> G[Collect Contributions]
    F2 --> G
    F3 --> G
    F4 --> G
    F5 --> G
    
    G --> H[Aggregator builds<br/>mdhn:StatusReport]
    H --> I[Return Brief Status Report<br/>to User]
    
    style A fill:#e1f5fe
    style H fill:#c8e6c9
    style I fill:#c8e6c9
    style E fill:#fff3e0
```


### Sequence view (more detailed):

```mermaid
sequenceDiagram
    participant U as User
    participant M as Map UI
    participant P as Processing Node
    participant O as Orchestrator
    participant S1 as Ownership Svc
    participant S2 as Authority Svc
    participant S3 as Other Subscribers
    participant A as Aggregator

    U->>M: Select Polygon on map
    M->>P: RegistrationPlateID + User Token
    P->>P: Authenticate
    P->>P: Load basic LandParcel data
    P->>O: Publish "ParcelSelected" event<br/>(RegistrationPlateID)
    
    par Parallel contributions
        O->>S1: Event
        S1-->>A: Owner profile(s) + shares
        O->>S2: Event
        S2-->>A: AuthorityReport
        O->>S3: Event
        S3-->>A: Area, Zoning, Status, ...
    end
    
    A->>A: Build StatusReport
    A->>P: Aggregated StatusReport
    P->>U: Brief report (Status of Land/Building)
```
### Processing Node + Message Broker (Publisher/Subscriber)
Here is a clean, focused diagram of the node we are discussing and its interaction with the message broker:

```mermaid
flowchart LR
    subgraph Client
        U[User / Map UI]
    end

    subgraph "Processing Node"
        A[1. Authenticate User]
        B[2. Resolve RegistrationPlateID<br/>پلاک ثبتی]
        C[3. Load basic LandParcel data]
        D[4. Publish Event]
        E[5. Wait for Aggregated Result]
        F[6. Return StatusReport to User]
    end

    subgraph "Message Broker (Pub/Sub)"
        EX[Exchange / Topic<br/>e.g. parcel.selected]
    end

    subgraph Subscribers
        S1[Ownership Service]
        S2[Authority Service]
        S3[Geometry / Area Service]
        S4[Zoning & District Service]
        S5[Permit / Status Service]
    end

    subgraph Aggregator
        AG[Aggregate Contributions<br/>→ mdhn:StatusReport]
    end

    U -->|Select parcel + token| A
    A --> B --> C --> D
    D -->|Publish<br/>RegistrationPlateID + context| EX

    EX -->|Fan-out| S1
    EX -->|Fan-out| S2
    EX -->|Fan-out| S3
    EX -->|Fan-out| S4
    EX -->|Fan-out| S5

    S1 -->|Contribution| AG
    S2 -->|Contribution| AG
    S3 -->|Contribution| AG
    S4 -->|Contribution| AG
    S5 -->|Contribution| AG

    AG -->|StatusReport| E
    E --> F
    F --> U
```
# UI proposal 

# Parcel selection & status report
*From map click to a Material status report, keyed by Registration Plate ID*

## 1. Purpose

This document proposes the first end-to-end screen pair for the Tehran Construction Ontology & Distributed Map-Based Platform: selecting a land parcel on the map, and viewing its aggregated status report. It follows the flow already defined in the DCWProposal architecture, where a map click resolves to a LandParcel via its Registration Plate ID (پلاک ثبتی), and a fan-out of subscriber services contributes ownership, authority, geometry, zoning, and permit data that an aggregator assembles into a single status report.

Two mockups are included: a low-fidelity wireframe for the exploration/selection screen, and a Material Design treatment for the resulting status report. Both are provided as standalone HTML files that render pixel-for-pixel and can be exported as PNG images (open the file in a browser and take a screenshot, or use a URL-to-image tool) or imported into Figma via any HTML-to-Figma plugin.

## 2. Recap of the underlying architecture

The platform's ontology treats Building Topology Ontology (BOT) as the physical-topology core, extended with an `mdhn:` module for districts, land parcels, zoning, documents, materials, policy, and workflow. The screens in this proposal sit on top of the "simple flow" already defined for parcel selection:

- User selects a polygon on the map.
- The client resolves that polygon to a LandParcel using its Registration Plate ID.
- The user is authenticated before anything proceeds; failure shows Access denied.
- A processing node publishes a `parcel.selected` event, keyed by the Registration Plate ID, to a message broker.
- Ownership, Authority, Geometry/Area, Zoning & District, and Permit/Status services subscribe and each contribute a fragment.
- An aggregator assembles the fragments into an `mdhn:StatusReport` and returns it to the user.

*The two screens below correspond to the first step (selection) and the last step (status report) of that flow.*

## 3. Screen 1 - Parcel selection (wireframe)

Deliberately kept low-fidelity so the team can agree on structure and interaction before investing in visual polish. Sketch-style annotations mark the parts of the flow that are backed by the pub/sub architecture rather than plain UI state.

![Fig. 1 - Wireframe sketch](images/parcel-selection-wireframe.png)
*Fig. 1 - Wireframe sketch: map with selectable parcels, search by Registration Plate ID, and the resulting selection panel.*

### Key elements

- **Search field** - looks up a parcel directly by Registration Plate ID, as an alternative to clicking the map.
- **Map canvas** - district boundary shown as a soft outline; individual parcels are tappable polygons; the active selection is highlighted.
- **Selection panel** - surfaces the resolved Registration Plate ID and district immediately, while area and further detail are marked pending until the status-report services respond.
- **Sign-in state** - shown persistently in the top bar; an annotation calls out that selection on an unauthenticated session should short-circuit to Access denied, per the architecture's authentication gate.
- **"Behind the click" checklist** - a design-only annotation, not shipped UI, tracing which of the five pipeline steps a given interaction has reached, to keep the mockup honest about latency (the aggregate step is not instant).

## 4. Screen 2 - Status report (Material view)

Once the aggregator returns a status report, the interface shifts from sketch exploration to a production-quality Material Design surface. Each card on the screen maps directly to one contributing service, which keeps the UI legible even as more services are added later.

![Fig. 2 - Material Design status report](images/parcel-status-report-material.png)
*Fig. 2 - Material Design status report for a selected parcel, assembled from the five subscriber services.*

### Section-to-service mapping

| Section on screen | Contributing service (per repo architecture) | Data shown |
|---|---|---|
| Parcel summary | Geometry/Area + Zoning & District services | Registration Plate ID, district, area, zoning class, overall status |
| Ownership | Ownership service | Owner names and ownership share |
| Permit / status | Permit/Status service | Workflow stepper (submitted → under review → approved → construction) |
| Geometry | Geometry/Area service | Area, perimeter, frontage |
| Authority | Authority service | Encumbrances, court holds, verification |
| Linked documents | Design & documentation layer (`mdhn:InformationResource`) | Geotechnical report, foundation design, BIM model, with approval state |

*Card-level tags (e.g. "Permit & status service") are shown as a design aid during this review and would likely be removed from the shipped UI once the aggregation is trusted end to end.*

## 5. Design rationale

- **Two fidelities, two purposes** - the wireframe is for agreeing on structure and data availability; the Material screen is for evaluating the shipped visual language against Tehran-municipality branding expectations.
- **Every card traces to a service** - this keeps the aggregator's contract visible in the UI and makes it easy to spot a missing or delayed contribution (see the "BIM model - Required" state in Fig. 2).
- **Status uses colour + icon, never colour alone** - the permit stepper and document states use both a check icon and a label so the report stays legible for colour-blind users and in print.
- **The wireframe encodes the pipeline, the Material screen hides it** - end users of the finished report should not need to know about the message broker; that context is deliberately confined to the design annotations.

## 6. Open questions / next steps

- Confirm the minimal set of mandatory statuses to show when a subscriber service times out (partial report vs. block until complete).
- Decide whether Registration Plate ID search and map-click selection should share one resolver endpoint or two.
- Extend the status report with a documents drawer that deep-links into the Design & Documentation layer (Site Plan, Architectural Plan, etc.).
- Validate the Material colour palette against Tehran Municipality's own visual identity guidelines, if one exists.
- Produce a Figma file from these HTML mockups (or rebuild natively in Figma) once the structure above is signed off, so component-level review and commenting can happen there.

---


The client’s (Business Expert) statement is showing other services. It moves the project from a focused “parcel status + permitting” tool to a **full Building Lifecycle platform**. This is a natural and powerful expansion of what we have already explained.

### How it fits with current foundation

The architecture we built is well-suited for this expansion:

- **Ontology as ubiquitous language** remains the correct center.  
- **Modular OWL extensions** allow us to add new domains without breaking existing ones.  
- **Publisher/Subscriber model** (via the message broker) is ideal for a lifecycle system, because each stage produces events that other stages can react to.

The current **Parcel Analysis → Status Report** flow (RegistrationPlateID / پلاک ثبتی) is actually the *correct first processing node*. It is the natural entry point into the entire lifecycle.

### Mapping the client’s lifecycle stages to Processing Nodes

Here is how I would interpret the Persian description in terms of our distributed pipeline:

| Client Stage (Persian)              | English                        | Likely Role in Pipeline                  | Notes |
|-------------------------------------|--------------------------------|------------------------------------------|-------|
| تحلیل پلاک                          | Parcel Analysis                | **Publisher + Aggregator** (current focus) | Entry point – already designed |
| اخذ پروانه                          | Permit Obtaining               | Publisher + Subscriber                   | Heavy rule-engine usage |
| طراحی                               | Design                         | Subscriber (BIM, documents) + Publisher  | Strong link to BOT + Documentation layer |
| ساخت                                | Construction                   | Publisher (progress events)              | Site updates, material consumption |
| مدیریت پروژه                        | Project Management             | Orchestrator / Coordinator               | Cross-cutting |
| تأمین مصالح                         | Material Supply                | Subscriber + Publisher                   | Links to Material layer |
| پرداخت‌ها                           | Payments                       | Subscriber (triggers) + Publisher        | Needs finance concepts |
| فروش                                | Sales / Transfer               | Publisher (ownership change)             | Critical for ownership history |
| نگهداری / تعمیرات                   | Maintenance / Repairs          | Subscriber + Publisher                   | Asset management phase |
| مدیریت دارایی                       | Asset Management               | Long-term Subscriber                     | Post-occupancy |
| ارتباط با بانک، بیمه، شهرداری...   | Bank, Insurance, Municipality… | External Subscribers / API Gateways      | Stakeholder integration |

Many of these will act as **both** Publishers and Subscribers depending on the moment in the lifecycle.

### Strategic Recommendation

1. **Keep the current scope as Phase 1**  
   Finish and validate the Parcel Analysis → Status Report vertical properly (ontology + Figma views + first services). This gives the client something concrete and valuable very quickly.

2. **Treat the full lifecycle as the product vision**  
   Document it clearly in the README as the long-term target. This shows the client that the architecture is not limited.

3. **Evolve the ontology modularly**  
   We will need new modules over time, for example:
   - `mdhn-lifecycle.ttl` (stages, transitions, milestones)
   - Stronger Stakeholder / Actor model (Bank, Insurer, Contractor, Supplier, etc.)
   - Financial concepts (Payment, Invoice, Guarantee…)
   - Asset & Maintenance concepts

4. **Define a Domain Event Catalog early**  
   The message broker should carry clear business events, e.g.:
   - `ParcelSelected`
   - `PermitApplicationSubmitted`
   - `PermitIssued`
   - `DesignApproved`
   - `ConstructionStarted`
   - `OwnershipTransferred`
   - `MaintenanceRequested`
   
   These events become the backbone of the distributed system.

### Overall Opinion

The client is describing a **Digital Construction Workplace / Building Lifecycle Platform**. This is ambitious but coherent. Our current ontology + map-centric + pub/sub foundation is a strong starting point precisely because it is modular and event-driven.

The biggest risk is scope creep. We should explicitly agree with the client on prioritization:

**Position of the Rule Engine in the Publisher/Subscriber pipeline**

The Rule Engine should be treated as a **first-class, specialized Processing Node** (a dedicated Rule Evaluation Service) rather than being buried inside every other service.

### Recommended placement

```
Event → Message Broker → Rule Evaluation Service → Decision Event → Orchestrator / Other Nodes
```

**Detailed flow:**

1. A Processing Node (e.g. Parcel Analysis or Permit Application) publishes a domain event  
   (`ParcelSelected`, `PermitApplicationSubmitted`, `DesignSubmitted`…).

2. The **Rule Evaluation Service** subscribes to these decision-relevant events.

3. It loads the current facts from the Knowledge Graph (the ontology instances for that `RegistrationPlateID` / Building / Application).

4. It selects and evaluates the applicable rules (filtered by district, building type, lifecycle stage, etc.).

5. It publishes a clear **Decision Event**:
   - `RulesEvaluationPassed` (Happy Path)
   - `RulesEvaluationFailed` (with structured reason codes)
   - `NeedsAdditionalInformation`
   - `Exception` (with suggested retry policy or escalation path)

6. The Orchestrator and other subscribers react to these decision events and determine the actual next step in the flow.

This design keeps rules **centralized, versioned, auditable, and consistent** — which is essential for municipal regulations that differ across Tehran’s 22 districts.

### Rule Ingestion Pipeline (PDF / DOCX → Executable Rules)

You correctly identified the real-world problem: most regulations arrive as PDF or DOCX.

**Recommended staged approach:**

| Stage | Description | Responsibility |
|-------|-------------|----------------|
| 1. Ingestion | Upload official PDF/DOCX circulars and codes | Admin / Domain expert |
| 2. Extraction | Text + structure extraction (layout-aware) | Semi-automated tool |
| 3. Intermediate Representation | Transform into a structured, human-readable rule format that supports AND / OR / NOT and references ontology terms | Transformation service + human review |
| 4. Validation & Approval | Legal/domain expert reviews and approves the formalized rules | Municipality expert |
| 5. Compilation | Convert intermediate form into the final engine’s language | Rule Engine adapter |
| 6. Activation | Versioned rule set becomes active for specific districts / stages | Rule Repository |

**For now (as you said):**  
Assume a flexible Rule Engine that can accept an intermediate language. Good practical choices for the intermediate layer:

- A controlled YAML / JSON rule DSL (easy to version and review)
- DMN decision tables (very readable for business experts)
- Or a combination of SHACL shapes + SPARQL for ontology-native constraints

The final engine can later be Drools, a lightweight custom engine, or even a hybrid (SHACL for structural constraints + a production rule engine for complex logic).

### How the Rule Engine determines Flows (Happy Path vs Exceptions)

The Rule Engine does **not** execute the whole business process. It **decides** and publishes the outcome. The Orchestrator (or a workflow engine) uses those decisions as gateways.

**Example decision outcomes:**

- **Happy Path**: All mandatory rules pass → continue to next lifecycle stage.
- **Soft Exception**: Missing document or temporary data issue → `NeedsAdditionalInformation` + retry policy (e.g., “retry in 30 minutes” or “notify applicant”).
- **Hard Exception**: Clear regulatory violation → route to manual review or rejection path.
- **Conditional Path**: Some rules pass only under certain conditions → different downstream branches.

Retry policies can be attached to specific exception codes (declared in the rules themselves or in a separate policy table). This keeps the retry logic declarative and auditable.

### Relation to the Ontology

Rules should be tightly bound to the ontology:

- Conditions reference ontology classes and properties (`mdhn:locatedInDistrict`, `mdhn:hasFoundationDesign`, height, zoning class, ownership status, etc.).
- This guarantees that the ubiquitous language used by business experts is the same language the rules use.
- When the ontology evolves, rules can be validated against it (e.g., via SHACL).

### Suggested high-level architecture diagram

```mermaid
flowchart TB
    subgraph Sources
        PDF[PDF / DOCX Regulations]
    end

    subgraph Rule Management
        TR[Transformation Service]
        IR[Intermediate Rules<br/>YAML / DMN / DSL]
        RR[Rule Repository<br/>Versioned + District-scoped]
    end

    subgraph Runtime
        MB[Message Broker]
        RES[Rule Evaluation Service]
        KG[Knowledge Graph<br/>Ontology Instances]
        ORCH[Orchestrator / Workflow]
    end

    PDF --> TR --> IR --> RR
    RR --> RES

    PN[Other Processing Nodes] -->|Domain Event| MB
    MB -->|Decision-relevant events| RES
    RES -->|Query facts| KG
    RES -->|Decision Event<br/>Passed / Failed / Exception / Retry| MB
    MB --> ORCH
    ORCH -->|Controls next Flow| PN
```

### Summary Recommendation

- Make the **Rule Evaluation Service** a dedicated subscriber that owns all regulatory decisions.
- Treat PDF/DOCX → Intermediate Representation as a first-class pipeline (with mandatory human approval).
- Let the Rule Engine **emit decisions**, not drive the entire process. The Orchestrator turns those decisions into Happy Paths, Exception Paths, and retries.
- Keep rules strongly linked to the ontology terms so the ubiquitous language stays consistent.

This gives us a clean separation of concerns while still allowing the Rule Engine to be the ultimate authority on whether a flow can proceed.

**Updated High-Level Architecture Diagram**  
Full Building Lifecycle as Processing Nodes (Publisher/Subscriber model)

```mermaid
flowchart TB
    %% ====================== PRESENTATION ======================
    subgraph Presentation["Presentation Layer"]
        UI[Map-Centric UI<br/>+ Dashboards]
        User[Users / Stakeholders]
    end

    %% ====================== CORE INFRASTRUCTURE ======================
    subgraph Core["Core Infrastructure"]
        MB[[Message Broker<br/>Pub/Sub Event Bus]]
        KG[(Knowledge Graph<br/>OWL Ontology + Instances)]
        RES[Rule Evaluation Service]
        ORCH[Orchestrator / Workflow Engine]
    end

    %% ====================== LIFECYCLE PROCESSING NODES ======================
    subgraph Lifecycle["Lifecycle Processing Nodes"]
        direction TB

        subgraph Pre["Pre-Construction"]
            PA[Parcel Analysis<br/>RegistrationPlateID]
            PERMIT[Permit Obtaining]
            DESIGN[Design<br/>BIM / Documents]
        end

        subgraph During["Construction Phase"]
            PM[Project Management]
            MAT[Material Supply]
            CONST[Construction<br/>Progress & Site]
            PAY[Payments]
        end

        subgraph Post["Post-Construction / Asset"]
            SALE[Sales / Ownership Transfer]
            MAINT[Maintenance]
            REPAIR[Repairs]
            ASSET[Asset Management]
        end
    end

    %% ====================== EXTERNAL STAKEHOLDERS ======================
    subgraph External["External Stakeholders & Gateways"]
        MUNI[Municipality Services]
        BANK[Bank / Finance]
        INS[Insurance]
        OTHER[Other Authorities]
    end

    %% ====================== CONNECTIONS ======================
    User --> UI
    UI <--> PA
    UI <--> PERMIT
    UI <--> DESIGN
    UI <--> ASSET

    %% All nodes talk through the broker
    PA --> MB
    PERMIT --> MB
    DESIGN --> MB
    PM --> MB
    MAT --> MB
    CONST --> MB
    PAY --> MB
    SALE --> MB
    MAINT --> MB
    REPAIR --> MB
    ASSET --> MB

    MB --> PA
    MB --> PERMIT
    MB --> DESIGN
    MB --> PM
    MB --> MAT
    MB --> CONST
    MB --> PAY
    MB --> SALE
    MB --> MAINT
    MB --> REPAIR
    MB --> ASSET

    %% Rule Engine & Orchestrator
    MB <--> RES
    RES <--> KG
    MB <--> ORCH
    ORCH <--> KG

    %% Knowledge Graph is the shared truth
    PA --> KG
    PERMIT --> KG
    DESIGN --> KG
    CONST --> KG
    SALE --> KG
    ASSET --> KG

    %% External systems
    MB <--> MUNI
    MB <--> BANK
    MB <--> INS
    MB <--> OTHER
```

### How to read this diagram

- **Presentation Layer**: The map-centric UI is the main entry point (especially for Parcel Analysis).
- **Core Infrastructure** (center):
  - **Message Broker** is the backbone of the distributed system.
  - **Knowledge Graph** (our BOT + mdhn ontology) is the single source of truth.
  - **Rule Evaluation Service** listens to events, evaluates rules, and publishes decisions (Happy Path / Exception / Retry).
  - **Orchestrator** reacts to decision events and coordinates longer-running flows.
- **Lifecycle Processing Nodes**: Each major stage from the client’s description is modeled as a Processing Node.  
  Nodes both **publish** domain events and **subscribe** to relevant events from other nodes.
- **External Stakeholders**: Connected via the same event bus (or through dedicated gateways).

### Key architectural principles illustrated

1. Every lifecycle stage is a Processing Node that communicates only through events.
2. The Rule Engine sits centrally and influences flow decisions without owning the entire process.
3. The Ontology/Knowledge Graph is continuously enriched by all nodes.
4. The system can grow stage-by-stage (start with Parcel Analysis + Permit, then add Design, Construction, etc.).

Would you like a second, more detailed version that also shows the main domain events flowing between these nodes (e.g. `ParcelSelected`, `PermitIssued`, `OwnershipTransferred`…)?



## Licence & Access

Private repository. Internal use only for the client project and authorised contributors.

---
*Reference: MehranDHN/DCWProposal - Tehran Construction Ontology & Distributed Map-Based Platform (private repository).*
