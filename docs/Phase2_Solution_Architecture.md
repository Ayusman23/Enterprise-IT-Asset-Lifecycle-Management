# Phase 2 — Solution Architecture
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Enterprise Architecture Overview

Built entirely on SAP BTP ABAP Environment following Clean Core: custom RAP business objects, no core modification, released APIs only.

```
┌─────────────────────────────────────────────────────────────┐
│                     SAP Fiori Launchpad                      │
│   (Asset Dashboard / Inventory / Maintenance / Object Page)  │
└───────────────────────────┬───────────────────────────────────┘
                             │ OData V4
┌───────────────────────────▼───────────────────────────────────┐
│                    SAP BTP ABAP Environment                   │
│  ┌───────────────┐ ┌────────────────┐ ┌────────────────────┐ │
│  │  Service Layer │ │  RAP Behavior   │ │  CDS View Layer     │ │
│  │                │ │  (Lifecycle FSM,│ │  (Interface/Proj/   │ │
│  │                │ │  Determinations)│ │   Consumption)      │ │
│  └───────────────┘ └────────┬───────┘ └────────────────────┘ │
│                              │                                  │
│                     ┌────────▼────────┐                        │
│                     │  SAP HANA Cloud │                        │
│                     └─────────────────┘                        │
└──────────────────────────────────────────────────────────────┘
```

## 2. SAP BTP Architecture

| Layer | Component |
|---|---|
| Identity | SAP Cloud Identity Services (IAS) |
| Development | SAP Business Application Studio |
| Runtime | SAP BTP ABAP Environment |
| UI Hosting | SAP Build Work Zone / Fiori Launchpad |
| Monitoring | SAP Cloud ALM |

## 3. RAP Architecture

- **Development model**: Managed RAP, draft-enabled root for asset registration (multi-field entry with validation before commit).
- Root entity: `Asset`. Children: `AssetAllocation`, `AssetMaintenance`, `AssetAuditLog`.
- Lifecycle status is modeled as a strict finite state machine enforced in behavior validations — this is the core architectural safeguard against illegal transitions like reallocating a retired asset.

## 4. Application Layers

| Layer | Responsibility |
|---|---|
| Interface CDS (`I_*`) | Reuse-focused source of truth |
| Projection CDS (`Z_C_*`) | Consumption/UI-facing, OData exposure |
| Behavior Definition/Implementation | Lifecycle state machine, validations, determinations, actions |
| Service Definition/Binding | OData V4 exposure |
| Fiori Elements | Asset Dashboard, Inventory Dashboard, Maintenance Dashboard |

## 5. Database Design (Summary — full detail in Phase 3)

Core tables: `ZASSET`, `ZASSET_CATEGORY`, `ZASSET_ALLOCATION`, `ZASSET_MAINTENANCE`, `ZASSET_AUDIT_LOG`, `ZEMPLOYEE`, `ZDEPARTMENT`.

## 6. Entity Relationship Diagram (textual)

```
AssetCategory (1) ───< (N) Asset
Asset (1) ───< (N) AssetAllocation
Asset (1) ───< (N) AssetMaintenance
Asset (1) ───< (N) AssetAuditLog
Employee (1) ───< (N) AssetAllocation
Department (1) ───< (N) Employee
```

## 7. Sequence Diagram — Allocate Asset

```
Asset Manager → Fiori App: Allocate Asset action
Fiori App → OData Service: POST /Asset(...)/allocate
OData Service → RAP Behavior: allocate action
RAP Behavior → Validation: check Status = 'IN_STOCK'
RAP Behavior → Determination: create AssetAllocation child, set Status = 'ALLOCATED'
RAP Behavior → Determination: create AssetAuditLog entry
RAP Behavior → OData Service: 200 OK, updated Asset
Fiori App → Asset Manager: confirmation, updated status badge
```

## 8. Folder Structure

```
project3-asset-lifecycle/
├── docs/
│   ├── Phase1_Business_Requirements.md
│   ├── Phase2_Solution_Architecture.md
│   ├── Phase3_Database_Design.md
│   ├── Phase4_CDS_Views.md
│   ├── Phase5_Behavior_Definitions.md
│   ├── Phase6_Behavior_Implementation.md
│   ├── Phase7_Service_Layer.md
│   ├── Phase8_SAP_Fiori_Elements.md
│   ├── Phase9_Authorization.md
│   ├── Phase10_Testing.md
│   ├── Phase11_Deployment.md
│   ├── Phase12_Documentation.md (see README.md)
│   └── README.md
├── src/
│   ├── cds/
│   │   ├── interface/
│   │   ├── projection/
│   │   └── metadata_extensions/
│   ├── behavior/
│   ├── classes/
│   ├── services/
│   └── fiori/
├── tests/
│   ├── unit/
│   └── integration/
├── images/
│   └── architecture/
└── configuration/
    └── future_enhancements.md
```

## 9. Architectural Decisions & Rationale

| Decision | Rationale |
|---|---|
| Draft-enabled Asset root | Registration requires multiple fields (serial, category, purchase data) entered safely before commit |
| Lifecycle as strict FSM in behavior validation | Prevents the exact audit-risk business rule stated in requirements: retired assets must never be reallocated |
| Separate `AssetAuditLog` child, system-write-only | Guarantees a tamper-evident, complete history independent of allocation/maintenance tables |
| `AssetMaintenance` decoupled from `AssetAllocation` | Maintenance scheduling is independent of who currently holds the asset — an asset in stock can still have scheduled maintenance |
| No AI layer in this project (unlike Project 2) | Scope explicitly excludes AI; future enhancements section documents where AI could be added later without redesign |

---

**Next:** Phase 3 — Database Design.
