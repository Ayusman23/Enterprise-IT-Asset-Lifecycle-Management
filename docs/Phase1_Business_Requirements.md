# Phase 1 — Business Requirement Analysis
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Business Requirements Document (BRD)

### 1.1 Purpose
This application governs the full lifecycle of IT assets — from registration through allocation, transfer, maintenance, return, and retirement — replacing manual spreadsheets with an auditable SAP BTP application.

### 1.2 Business Problem
- No single source of truth for who holds which asset.
- Maintenance is reactive, not scheduled — leads to downtime and compliance gaps.
- Asset transfers between employees/departments are undocumented.
- Retired assets sometimes get reallocated by mistake, creating audit findings.

### 1.3 Business Goals
| Goal | Description |
|---|---|
| G1 | Single governed system of record for every IT asset |
| G2 | Enforce lifecycle state integrity (no reallocating retired assets) |
| G3 | Enable preventive maintenance scheduling and history |
| G4 | Provide inventory and department dashboards for planning |
| G5 | Deliver a clean-core, upgrade-safe SAP BTP solution |

---

## 2. Functional Requirements

| ID | Requirement |
|---|---|
| FR-01 | IT Administrator can register a new asset with category, serial number, and purchase details |
| FR-02 | System enforces a unique Asset ID per asset |
| FR-03 | Asset Manager can allocate an asset to an employee |
| FR-04 | System records allocation date, assigning administrator, and condition at allocation |
| FR-05 | Asset Manager can transfer an asset between employees/departments with authorization |
| FR-06 | Employee/Administrator can initiate an asset return |
| FR-07 | IT Administrator can schedule preventive maintenance with recurrence |
| FR-08 | System logs every maintenance activity with technician, date, and outcome |
| FR-09 | Asset Manager can retire an asset, closing the lifecycle |
| FR-10 | Retired assets cannot be allocated or transferred (hard system block) |
| FR-11 | System supports search, filter, and sort across all asset list views |
| FR-12 | System generates inventory and department utilization reports |
| FR-13 | Every lifecycle-changing action is captured in an audit trail |
| FR-14 | Dashboard shows assets due for maintenance in the next 30 days |
| FR-15 | System supports asset disposal recording (final state after retirement) |

## 3. Non-Functional Requirements

| ID | Category | Requirement |
|---|---|---|
| NFR-01 | Performance | Asset list must return in < 2s for up to 100,000 asset records |
| NFR-02 | Availability | 99.5% uptime during business hours |
| NFR-03 | Security | RBAC enforced at CDS and RAP behavior level |
| NFR-04 | Scalability | Multi-department, multi-location data model |
| NFR-05 | Maintainability | 100% Clean Core — no core modifications |
| NFR-06 | Auditability | Every state transition logged with timestamp, actor, and reason |
| NFR-07 | Usability | Fiori Elements responsive across desktop, tablet, mobile |
| NFR-08 | Data Integrity | Lifecycle state machine strictly enforced — no illegal transitions |

## 4. Actors

| Actor | Description |
|---|---|
| Employee | Holds allocated assets, initiates returns |
| IT Administrator | Registers assets, schedules maintenance |
| Asset Manager | Allocates, transfers, retires assets, owns lifecycle decisions |
| System Administrator | Manages roles, configuration |

## 5. User Stories (Sample)

- **US-01**: As an IT Administrator, I want to register a new laptop with serial number and category so it enters the tracked inventory.
- **US-02**: As an Asset Manager, I want to allocate an asset to an employee so ownership is recorded.
- **US-03**: As an IT Administrator, I want to schedule recurring preventive maintenance so assets don't fail unexpectedly.
- **US-04**: As an Asset Manager, I want the system to block allocation of a retired asset so audit integrity is preserved.
- **US-05**: As an Employee, I want to see all assets currently allocated to me.

## 6. Business Rules

| ID | Rule |
|---|---|
| BR-01 | Each asset must have a unique Asset ID |
| BR-02 | Every asset belongs to exactly one category |
| BR-03 | Every asset has exactly one lifecycle status at any time |
| BR-04 | Every allocation must be recorded with date and responsible administrator |
| BR-05 | Every maintenance activity must be logged, including outcome |
| BR-06 | Asset transfers require Asset Manager authorization |
| BR-07 | Retired assets cannot be reallocated or transferred |
| BR-08 | All critical operations (allocate, transfer, retire, dispose) must be audited |

## 7. Business Workflow

```
IT Administrator registers asset (Registered)
        ↓
Asset validation
        ↓
Asset allocation (Allocated) → Employee assignment
        ↓
Preventive maintenance scheduling (recurring, independent of status)
        ↓
Asset transfer (if required) → remains Allocated, new owner
        ↓
Asset return (Returned / In Stock)
        ↓
Asset retirement (Retired)
        ↓
Asset disposal (Disposed)
```

## 8. Acceptance Criteria

- AC-01: Given a unique serial number, when an asset is registered, then it enters status `Registered` / `In Stock`.
- AC-02: Given an asset is `In Stock`, when allocated to an employee, then status becomes `Allocated` and an allocation record is created.
- AC-03: Given an asset is `Retired`, when an allocation is attempted, then the system blocks it with a clear error.
- AC-04: Given a maintenance schedule exists, when the due date arrives, then the asset appears on the "Due for Maintenance" dashboard list.
- AC-05: Given an asset is returned, when the return is processed, then status becomes `In Stock` and prior allocation is closed with a return date.

---

**Next:** Phase 2 — Solution Architecture.
