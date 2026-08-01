# Combined Documentation

> This file consolidates all documents generated for Project 3 (Enterprise IT Asset Lifecycle Management): full Phase 1-12 docs.

---

# Project 3 - Enterprise IT Asset Lifecycle Management (Phase 1-12 Docs)

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

---

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

---

# Phase 3 — Database Design
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Naming Standards
All custom tables use prefix `Z`, per SAP naming conventions for ABAP Cloud (customer namespace).

## 2. Tables

### 2.1 ZASSET_CATEGORY
| Field | Type | Key | Description |
|---|---|---|---|
| CATEGORY_ID | CHAR(6) | PK | e.g. LAPTOP, MONITOR, PHONE |
| CATEGORY_NAME | CHAR(40) | | Display name |
| DEFAULT_MAINT_CYCLE_DAYS | INT4 | | Default preventive maintenance interval |

### 2.2 ZASSET (Root, Draft-enabled)
| Field | Type | Key | Description |
|---|---|---|---|
| ASSET_UUID | RAW(16) | PK | RAP technical key |
| ASSET_ID | CHAR(12) | | Unique business Asset ID |
| SERIAL_NUMBER | CHAR(40) | | Manufacturer serial number (unique) |
| CATEGORY_ID | CHAR(6) | FK → ZASSET_CATEGORY | Category |
| MODEL | CHAR(60) | | Model description |
| PURCHASE_DATE | DATS | | Acquisition date |
| PURCHASE_VALUE | CURR(13,2) | | Acquisition cost |
| STATUS | CHAR(4) | | RG/IS/AL/TR/RE/DI (Registered/InStock/Allocated/Retired/Disposed) |
| CURRENT_EMPLOYEE_ID | CHAR(10) | FK → ZEMPLOYEE, nullable | Current holder, null if in stock |
| CREATED_BY | CHAR(12) | | Audit |
| CREATED_AT | TIMESTAMPL | | Audit |
| LAST_CHANGED_BY | CHAR(12) | | Audit |
| LAST_CHANGED_AT | TIMESTAMPL | | Audit |

### 2.3 ZASSET_ALLOCATION (Child)
| Field | Type | Key | Description |
|---|---|---|---|
| ASSET_UUID | RAW(16) | PK, FK → ZASSET | Parent |
| ALLOCATION_UUID | RAW(16) | PK | Row key |
| EMPLOYEE_ID | CHAR(10) | FK → ZEMPLOYEE | Recipient |
| ALLOCATED_BY | CHAR(12) | | Administrator who allocated |
| ALLOCATED_AT | TIMESTAMPL | | Allocation timestamp |
| RETURNED_AT | TIMESTAMPL | nullable | Return timestamp, null if still held |
| CONDITION_AT_ALLOCATION | CHAR(10) | | New/Good/Fair/Poor |
| CONDITION_AT_RETURN | CHAR(10) | nullable | Assessed at return |

### 2.4 ZASSET_MAINTENANCE (Child)
| Field | Type | Key | Description |
|---|---|---|---|
| ASSET_UUID | RAW(16) | PK, FK → ZASSET | Parent |
| MAINTENANCE_UUID | RAW(16) | PK | Row key |
| SCHEDULED_DATE | DATS | | Planned maintenance date |
| RECURRENCE_DAYS | INT4 | | Repeat interval, 0 if one-time |
| PERFORMED_DATE | DATS | nullable | Actual date performed |
| TECHNICIAN_ID | CHAR(12) | nullable | Who performed it |
| OUTCOME | CHAR(10) | nullable | OK/Repaired/Flagged |
| NOTES | CHAR(255) | | Free text |

### 2.5 ZASSET_AUDIT_LOG (Child, system-write-only)
| Field | Type | Key | Description |
|---|---|---|---|
| ASSET_UUID | RAW(16) | PK, FK → ZASSET | Parent |
| LOG_UUID | RAW(16) | PK | Row key |
| ACTION | CHAR(12) | | REGISTERED/ALLOCATED/TRANSFERRED/RETURNED/RETIRED/DISPOSED |
| ACTOR_ID | CHAR(12) | | Who performed the action |
| DETAILS | CHAR(255) | | Free-text context |
| ACTION_AT | TIMESTAMPL | | Timestamp |

## 3. Relationships

- `ZASSET.CATEGORY_ID` → `ZASSET_CATEGORY.CATEGORY_ID` (N:1)
- `ZASSET.CURRENT_EMPLOYEE_ID` → `ZEMPLOYEE.EMPLOYEE_ID` (N:1, nullable)
- `ZASSET_ALLOCATION.ASSET_UUID` → `ZASSET.ASSET_UUID` (N:1, composition)
- `ZASSET_ALLOCATION.EMPLOYEE_ID` → `ZEMPLOYEE.EMPLOYEE_ID` (N:1)
- `ZASSET_MAINTENANCE.ASSET_UUID` → `ZASSET.ASSET_UUID` (N:1, composition)
- `ZASSET_AUDIT_LOG.ASSET_UUID` → `ZASSET.ASSET_UUID` (N:1, composition)

## 4. Indexes

| Table | Index | Purpose |
|---|---|---|
| ZASSET | IDX_STATUS_CAT (STATUS, CATEGORY_ID) | Fast inventory dashboard filtering |
| ZASSET | IDX_SERIAL (SERIAL_NUMBER) unique | Enforce serial uniqueness, fast lookup |
| ZASSET_ALLOCATION | IDX_EMP (EMPLOYEE_ID, RETURNED_AT) | "My assets" / active allocation lookup |
| ZASSET_MAINTENANCE | IDX_DUE (SCHEDULED_DATE, PERFORMED_DATE) | "Due for maintenance" dashboard query |

## 5. Business Rules Enforced at DB/Behavior Level

- `STATUS` transitions constrained to the defined lifecycle state machine (Phase 5 validations).
- An `Asset` with `STATUS = 'RE'` (Retired) or `'DI'` (Disposed) cannot receive a new `ZASSET_ALLOCATION` row — enforced in the `allocate` action's precondition, not just UI.
- `SERIAL_NUMBER` uniqueness enforced via unique index, not application-only check, to prevent race conditions.
- `ZASSET_ALLOCATION.RETURNED_AT` must be null for at most one row per asset at any time (only one active holder).

## 6. Normalization

The model is in **3NF**. Current holder (`CURRENT_EMPLOYEE_ID`) is denormalized onto `ZASSET` intentionally as a fast-access cache for list views, but the authoritative allocation history lives in `ZASSET_ALLOCATION` — kept in sync exclusively via the behavior implementation's determinations, never written directly by any other process, avoiding update anomalies.

---

**Next:** Phase 4 — CDS Views.

---

# Phase 4 — CDS Views
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Interface View — Asset

```abap
@AbapCatalog.sqlViewName: 'ZIASSET'
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Asset - Interface'
define root view entity ZI_Asset
  as select from zasset
  composition [0..*] of ZI_AssetAllocation as _Allocation
  composition [0..*] of ZI_AssetMaintenance as _Maintenance
  composition [0..*] of ZI_AssetAuditLog as _AuditLog
  association [1..1] to ZI_AssetCategory as _Category on $projection.CategoryId = _Category.CategoryId
  association [0..1] to ZI_Employee as _CurrentHolder on $projection.CurrentEmployeeId = _CurrentHolder.EmployeeId
{
  key asset_uuid           as AssetUUID,
      asset_id              as AssetId,
      serial_number          as SerialNumber,
      category_id            as CategoryId,
      model                  as Model,
      purchase_date          as PurchaseDate,
      purchase_value         as PurchaseValue,
      status                 as Status,
      current_employee_id    as CurrentEmployeeId,
      created_by             as CreatedBy,
      created_at             as CreatedAt,
      last_changed_by        as LastChangedBy,
      last_changed_at        as LastChangedAt,

      _Category,
      _CurrentHolder,
      _Allocation,
      _Maintenance,
      _AuditLog
}
```

## 2. Interface View — Maintenance

```abap
@AbapCatalog.sqlViewName: 'ZIASSETMAINT'
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Asset Maintenance - Interface'
define view entity ZI_AssetMaintenance
  as select from zasset_maintenance
{
  key maintenance_uuid as MaintenanceUUID,
      asset_uuid         as AssetUUID,
      scheduled_date     as ScheduledDate,
      recurrence_days    as RecurrenceDays,
      performed_date     as PerformedDate,
      technician_id      as TechnicianId,
      outcome            as Outcome,
      notes              as Notes
}
```

## 3. Projection View — Asset (Consumption)

```abap
@AbapCatalog.sqlViewName: 'ZCASSET'
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'IT Asset'
@Metadata.allowExtensions: true
@Search.searchable: true
define root view entity Z_C_Asset
  provider contract transactional_query
  as projection on ZI_Asset as Asset
{
  key AssetUUID,
      @Search.defaultSearchElement: true
      AssetId,
      @Search.defaultSearchElement: true
      SerialNumber,
      @ObjectModel.text.element: ['CategoryName']
      CategoryId,
      _Category.CategoryName as CategoryName : localized,
      Model,
      PurchaseDate,
      PurchaseValue,
      @ObjectModel.text.element: ['StatusText']
      Status,
      @EndUserText.label: 'Status Text'
      case Status
        when 'RG' then 'Registered'
        when 'IS' then 'In Stock'
        when 'AL' then 'Allocated'
        when 'RE' then 'Retired'
        when 'DI' then 'Disposed'
        else 'Unknown'
      end as StatusText : String(20),
      @ObjectModel.text.element: ['CurrentHolderName']
      CurrentEmployeeId,
      _CurrentHolder.FullName as CurrentHolderName : localized,
      CreatedBy,
      CreatedAt,
      LastChangedBy,
      LastChangedAt,

      _Category : redirected to Z_C_AssetCategory,
      _CurrentHolder : redirected to Z_C_Employee,
      _Allocation : redirected to composition child Z_C_AssetAllocation,
      _Maintenance : redirected to composition child Z_C_AssetMaintenance,
      _AuditLog : redirected to composition child Z_C_AssetAuditLog
}
```

## 4. Metadata Extension — List Report / Object Page Annotations

```abap
@Metadata.layer: #CORE
@UI: {
  headerInfo: {
    typeName: 'Asset',
    typeNamePlural: 'Assets',
    title: { type: #STANDARD, value: 'AssetId' }
  }
}
annotate view Z_C_Asset with
{
  @UI.facet: [
    { id: 'GeneralInfo', purpose: #STANDARD, type: #IDENTIFICATION_REFERENCE, label: 'General Information', position: 10 },
    { id: 'Maintenance', purpose: #STANDARD, type: #LINEITEM_REFERENCE, targetElement: '_Maintenance', label: 'Maintenance History', position: 20 },
    { id: 'AuditLog', purpose: #STANDARD, type: #LINEITEM_REFERENCE, targetElement: '_AuditLog', label: 'Audit Trail', position: 30 }
  ]

  @UI.lineItem: [{ position: 10, importance: #HIGH }]
  @UI.identification: [{ position: 10 }]
  AssetId;

  @UI.lineItem: [{ position: 20 }]
  @UI.identification: [{ position: 20 }]
  SerialNumber;

  @UI.lineItem: [{ position: 30 }]
  @UI.identification: [{ position: 30 }]
  CategoryName;

  @UI.lineItem: [{ position: 40, criticality: 'StatusCriticality' }]
  @UI.identification: [{ position: 40 }]
  StatusText;

  @UI.lineItem: [{ position: 50 }]
  @UI.identification: [{ position: 50 }]
  CurrentHolderName;
}
```

## 5. Consumption View — Inventory Dashboard (Analytical)

```abap
@AbapCatalog.sqlViewName: 'ZCINVENTORY'
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Inventory Summary by Category and Status'
@Analytics.query: true
define view entity Z_C_InventorySummary
  as select from ZI_Asset as Asset
{
  key Asset.CategoryId,
  key Asset.Status,
      @Aggregation.default: #COUNT
      Asset.AssetUUID as AssetCount
}
group by Asset.CategoryId, Asset.Status
```

## 6. Consumption View — Due for Maintenance

```abap
@AbapCatalog.sqlViewName: 'ZCMAINTDUE'
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Assets Due for Maintenance'
define view entity Z_C_MaintenanceDue
  as select from ZI_AssetMaintenance as Maint
  inner join ZI_Asset as Asset on Maint.AssetUUID = Asset.AssetUUID
  where Maint.ScheduledDate <= $session.system_date + 30
    and Maint.PerformedDate is initial
{
  key Maint.MaintenanceUUID,
      Asset.AssetId,
      Asset.Model,
      Maint.ScheduledDate
}
```

## 7. Annotation Rationale

| Annotation | Why |
|---|---|
| `@ObjectModel.text.element` | Renders technical keys (CategoryId, Status, EmployeeId) as readable text |
| `criticality: 'StatusCriticality'` | Color-codes lifecycle status (e.g. red for Retired) |
| `@Analytics.query` | Enables KPI tiles and SAC consumption for inventory summary |
| `where` clause on `Z_C_MaintenanceDue` | Pushes the 30-day due-date filter to the database layer for dashboard performance |

---

**Next:** Phase 5 — Behavior Definitions.

---

# Phase 5 — Behavior Definitions
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Root Behavior Definition — Asset

```abap
managed implementation in class zbp_i_asset unique;
strict ( 2 );

define behavior for ZI_Asset alias Asset
persistent table zasset
draft table zasset_d
lock master
authorization master ( instance )
etag master last_changed_at
{
  create;
  update( features : instance );
  delete( features : instance );

  field ( readonly ) AssetUUID, AssetId, CreatedBy, CreatedAt, LastChangedBy, LastChangedAt, Status, CurrentEmployeeId;
  field ( mandatory ) SerialNumber, CategoryId, Model, PurchaseDate;

  action ( features : instance ) allocate parameter ZA_AllocateParams result [1] $self;
  action ( features : instance ) transfer parameter ZA_TransferParams result [1] $self;
  action ( features : instance ) returnAsset result [1] $self;
  action ( features : instance ) retire parameter ZA_RetireParams result [1] $self;
  action ( features : instance ) dispose parameter ZA_DisposeParams result [1] $self;
  action ( features : instance ) scheduleMaintenance parameter ZA_ScheduleMaintParams result [1] $self;

  determination setInitialStatus on modify { create; }
  determination logRegistration on save { create; }
  determination logLifecycleChange on save { field Status; }

  validation validateUniqueSerial on save { field SerialNumber; }
  validation validateStatusTransition on save { field Status; }
  validation validateAllocationTarget on save { field CurrentEmployeeId; }

  association _Allocation   { create; }
  association _Maintenance  { create; }
  association _AuditLog     { create; }

  mapping for zasset
  {
    AssetUUID          = asset_uuid;
    AssetId             = asset_id;
    SerialNumber        = serial_number;
    CategoryId           = category_id;
    Model                = model;
    PurchaseDate         = purchase_date;
    PurchaseValue        = purchase_value;
    Status               = status;
    CurrentEmployeeId    = current_employee_id;
    CreatedBy            = created_by;
    CreatedAt            = created_at;
    LastChangedBy        = last_changed_by;
    LastChangedAt        = last_changed_at;
  }
}
```

## 2. Draft Handling

```abap
define behavior for ZI_Asset alias Asset
...
{
  ...
  draft action Activate optimized;
  draft action Discard;
  draft action Resume;
  draft action Edit;
  draft determine action Prepare;
}
```
Draft supports the multi-field registration flow (serial number lookup/validation, category selection, purchase details) before the asset formally enters the `Registered` / `In Stock` state.

## 3. Child Behavior Definitions

### 3.1 Allocation (append-only, closed by returnAsset action)
```abap
managed implementation in class zbp_i_assetallocation unique;
strict ( 2 );

define behavior for ZI_AssetAllocation alias AssetAllocation
persistent table zasset_allocation
lock dependent by _Parent
authorization dependent by _Parent
{
  update( features : instance ) [ internal ];
  field ( readonly ) AllocationUUID, EmployeeId, AllocatedBy, AllocatedAt, ReturnedAt, ConditionAtAllocation, ConditionAtReturn;
  association _Parent { }

  mapping for zasset_allocation
  {
    AllocationUUID          = allocation_uuid;
    AssetUUID                 = asset_uuid;
    EmployeeId                 = employee_id;
    AllocatedBy                = allocated_by;
    AllocatedAt                = allocated_at;
    ReturnedAt                  = returned_at;
    ConditionAtAllocation        = condition_at_allocation;
    ConditionAtReturn            = condition_at_return;
  }
}
```

### 3.2 Maintenance
```abap
managed implementation in class zbp_i_assetmaintenance unique;
strict ( 2 );

define behavior for ZI_AssetMaintenance alias AssetMaintenance
persistent table zasset_maintenance
lock dependent by _Parent
authorization dependent by _Parent
{
  update( features : instance );
  field ( readonly ) MaintenanceUUID;
  field ( mandatory ) ScheduledDate;

  action ( features : instance ) completeMaintenance parameter ZA_CompleteMaintParams result [1] $self;

  association _Parent { }

  mapping for zasset_maintenance
  {
    MaintenanceUUID   = maintenance_uuid;
    AssetUUID           = asset_uuid;
    ScheduledDate         = scheduled_date;
    RecurrenceDays         = recurrence_days;
    PerformedDate           = performed_date;
    TechnicianId             = technician_id;
    Outcome                   = outcome;
    Notes                     = notes;
  }
}
```

### 3.3 Audit Log (system-write-only)
```abap
managed implementation in class zbp_i_assetauditlog unique;
strict ( 2 );

define behavior for ZI_AssetAuditLog alias AssetAuditLog
persistent table zasset_audit_log
lock dependent by _Parent
authorization dependent by _Parent
{
  field ( readonly ) LogUUID, Action, ActorId, Details, ActionAt;
  association _Parent { }

  mapping for zasset_audit_log
  {
    LogUUID     = log_uuid;
    AssetUUID    = asset_uuid;
    Action        = action;
    ActorId        = actor_id;
    Details         = details;
    ActionAt         = action_at;
  }
}
```
No `create`/`update` operations are exposed externally on `AssetAuditLog` — rows are only inserted internally by the parent's determinations, guaranteeing tamper-evident history.

## 4. Actions

| Action | Purpose | Precondition |
|---|---|---|
| `allocate` | In Stock → Allocated | `Status = 'IS'` |
| `transfer` | Allocated → Allocated (new holder) | `Status = 'AL'`, Asset Manager authorization |
| `returnAsset` | Allocated → In Stock | `Status = 'AL'` |
| `retire` | In Stock/Allocated → Retired | current holder must be cleared first if Allocated |
| `dispose` | Retired → Disposed | `Status = 'RE'` |
| `scheduleMaintenance` | (any status) → creates `AssetMaintenance` child | none — maintenance is status-independent |

## 5. Determinations

| Determination | Trigger | Purpose |
|---|---|---|
| `setInitialStatus` | on create | Sets Status = 'RG' then immediately 'IS' after registration validation passes |
| `logRegistration` | on save, after create | Writes initial `AssetAuditLog` entry |
| `logLifecycleChange` | on save, when Status changes | Writes an `AssetAuditLog` entry for every transition |

## 6. Validations

| Validation | Rule Enforced |
|---|---|
| `validateUniqueSerial` | No two active assets share the same `SerialNumber` |
| `validateStatusTransition` | Only legal FSM transitions permitted (see Phase 6 state table) |
| `validateAllocationTarget` | `CurrentEmployeeId` must reference an active employee when `Status = 'AL'` |

## 7. Locking & Authorization
`lock master` with ETag on `LastChangedAt` prevents concurrent transfer/retire race conditions. `authorization master (instance)` delegates to the behavior implementation, checking Asset Manager role against the asset's department scope (Phase 9).

---

**Next:** Phase 6 — Behavior Implementation.

---

# Phase 6 — Behavior Implementation
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Lifecycle State Machine

| From | Action | To | Guard |
|---|---|---|---|
| (new) | create | RG (Registered) | mandatory fields present |
| RG | (auto determination) | IS (In Stock) | unique serial validated |
| IS | allocate | AL (Allocated) | target employee active |
| AL | transfer | AL (new holder) | Asset Manager role |
| AL | returnAsset | IS | none |
| IS / AL | retire | RE | if AL, holder cleared first (system-forced return) |
| RE | dispose | DI | Asset Manager role |
| RE / DI | allocate / transfer | **blocked** | hard validation failure |

## 2. Validation — Status Transition Guard

```abap
CLASS lhc_Asset IMPLEMENTATION.

  METHOD validateStatusTransition.
    READ ENTITIES OF zi_asset IN LOCAL MODE
      ENTITY Asset
        FIELDS ( Status )
        WITH CORRESPONDING #( keys )
      RESULT DATA(assets).

    LOOP AT assets INTO DATA(asset).
      " Load previous persisted status for comparison
      SELECT SINGLE status FROM zasset WHERE asset_uuid = @asset-AssetUUID INTO @DATA(lv_old_status).

      IF zcl_asset_fsm=>is_valid_transition( iv_from = lv_old_status iv_to = asset-Status ) = abap_false.
        APPEND VALUE #( %tky = asset-%tky
                         %msg = NEW zcx_asset_messages( textid = zcx_asset_messages=>illegal_transition
                                                          from_status = lv_old_status
                                                          to_status   = asset-Status ) )
               TO reported-asset.
        APPEND VALUE #( %tky = asset-%tky ) TO failed-asset.
      ENDIF.
    ENDLOOP.
  ENDMETHOD.

ENDCLASS.
```

```abap
CLASS zcl_asset_fsm DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    CLASS-METHODS is_valid_transition
      IMPORTING iv_from TYPE char4
                iv_to   TYPE char4
      RETURNING VALUE(rv_valid) TYPE abap_boolean.
ENDCLASS.

CLASS zcl_asset_fsm IMPLEMENTATION.
  METHOD is_valid_transition.
    DATA: lt_allowed TYPE HASHED TABLE OF char8 WITH UNIQUE KEY table_line.
    lt_allowed = VALUE #( ( |RGIS| ) ( |ISAL| ) ( |ALAL| ) ( |ALIS| )
                           ( |ISRE| ) ( |ALRE| ) ( |REDI| ) ).
    rv_valid = xsdbool( line_exists( lt_allowed[ table_line = |{ iv_from }{ iv_to }| ] ) ).
  ENDMETHOD.
ENDCLASS.
```

## 3. Action Implementation — Allocate

```abap
  METHOD allocate.
    READ ENTITIES OF zi_asset IN LOCAL MODE
      ENTITY Asset FIELDS ( Status ) WITH CORRESPONDING #( keys )
      RESULT DATA(assets).

    LOOP AT keys INTO DATA(key).
      DATA(asset) = assets[ KEY entity AssetUUID = key-AssetUUID ].

      IF asset-Status <> 'IS'.
        APPEND VALUE #( %tky = key-%tky
                         %msg = NEW zcx_asset_messages( textid = zcx_asset_messages=>not_in_stock ) )
               TO reported-asset.
        APPEND VALUE #( %tky = key-%tky ) TO failed-asset.
        CONTINUE.
      ENDIF.

      MODIFY ENTITIES OF zi_asset IN LOCAL MODE
        ENTITY Asset
          UPDATE FIELDS ( Status CurrentEmployeeId )
            WITH VALUE #( ( %tky = key-%tky Status = 'AL' CurrentEmployeeId = key-%param-EmployeeId ) )
          CREATE BY \_Allocation
            FIELDS ( EmployeeId AllocatedBy AllocatedAt ConditionAtAllocation )
            WITH VALUE #( ( %tky    = key-%tky
                             %target = VALUE #( ( %cid = 'AL1'
                                                   EmployeeId             = key-%param-EmployeeId
                                                   AllocatedBy            = cl_abap_context_info=>get_user_technical_name( )
                                                   AllocatedAt            = utclong_current( )
                                                   ConditionAtAllocation  = key-%param-Condition ) ) ) )
        FAILED failed
        REPORTED reported.
    ENDLOOP.

    READ ENTITIES OF zi_asset IN LOCAL MODE ENTITY Asset ALL FIELDS WITH CORRESPONDING #( keys )
      RESULT DATA(updated).
    result = VALUE #( FOR u IN updated ( %tky = u-%tky %param = u ) ).
  ENDMETHOD.
```

## 4. Action Implementation — Retire (with forced return)

```abap
  METHOD retire.
    READ ENTITIES OF zi_asset IN LOCAL MODE
      ENTITY Asset FIELDS ( Status ) WITH CORRESPONDING #( keys )
      RESULT DATA(assets).

    LOOP AT keys INTO DATA(key).
      DATA(asset) = assets[ KEY entity AssetUUID = key-AssetUUID ].

      IF asset-Status = 'AL'.
        " Force-close any open allocation before retiring
        MODIFY ENTITIES OF zi_asset IN LOCAL MODE
          ENTITY Asset UPDATE BY \_Allocation
            FIELDS ( ReturnedAt )
            WITH VALUE #( ( %tky = key-%tky %target = VALUE #( ( ReturnedAt = utclong_current( ) ) ) ) ).
      ENDIF.

      MODIFY ENTITIES OF zi_asset IN LOCAL MODE
        ENTITY Asset
          UPDATE FIELDS ( Status CurrentEmployeeId )
          WITH VALUE #( ( %tky = key-%tky Status = 'RE' CurrentEmployeeId = '' ) )
        FAILED failed
        REPORTED reported.
    ENDLOOP.

    READ ENTITIES OF zi_asset IN LOCAL MODE ENTITY Asset ALL FIELDS WITH CORRESPONDING #( keys )
      RESULT DATA(updated).
    result = VALUE #( FOR u IN updated ( %tky = u-%tky %param = u ) ).
  ENDMETHOD.
```

## 5. Determination — Audit Log Writer

```abap
  METHOD logLifecycleChange.
    READ ENTITIES OF zi_asset IN LOCAL MODE
      ENTITY Asset FIELDS ( Status ) WITH CORRESPONDING #( keys )
      RESULT DATA(assets).

    LOOP AT assets INTO DATA(asset).
      MODIFY ENTITIES OF zi_asset IN LOCAL MODE
        ENTITY Asset
          CREATE BY \_AuditLog
            FIELDS ( Action ActorId Details ActionAt )
            WITH VALUE #( ( %tky    = asset-%tky
                             %target = VALUE #( ( %cid    = 'LOG1'
                                                   Action    = asset-Status
                                                   ActorId   = cl_abap_context_info=>get_user_technical_name( )
                                                   Details   = |Status changed to { asset-Status }|
                                                   ActionAt  = utclong_current( ) ) ) ) ).
    ENDLOOP.
  ENDMETHOD.
```

## 6. Error Handling & Messages

Business exceptions use `ZCX_ASSET_MESSAGES` extending `CX_ABAP_BEHV` with typed text IDs (`ILLEGAL_TRANSITION`, `NOT_IN_STOCK`, `DUPLICATE_SERIAL`, `RETIRED_ASSET_BLOCKED`) — always translatable message texts, never hardcoded strings, consistent with SAP Fiori Elements error message presentation standards.

---

**Next:** Phase 7 — Service Layer.

---

# Phase 7 — Service Layer
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Service Definition

```abap
@EndUserText.label: 'IT Asset Lifecycle Management Service'
define service Z_UI_ASSET_LIFECYCLE_O4 {
  expose Z_C_Asset as Asset;
  expose Z_C_AssetAllocation as AssetAllocation;
  expose Z_C_AssetMaintenance as AssetMaintenance;
  expose Z_C_AssetAuditLog as AssetAuditLog;
  expose Z_C_AssetCategory as AssetCategory;
  expose Z_C_Employee as Employee;
  expose Z_C_InventorySummary as InventorySummary;
  expose Z_C_MaintenanceDue as MaintenanceDue;
}
```

## 2. Service Binding

| Property | Value |
|---|---|
| Binding Name | `Z_UI_ASSET_LIFECYCLE_O4` |
| Binding Type | OData V4 - UI |
| Service Definition | `Z_UI_ASSET_LIFECYCLE_O4` |
| Published Path | `/sap/opu/odata4/sap/z_ui_asset_lifecycle_o4/srvd/sap/z_ui_asset_lifecycle_o4/0001/` |

## 3. Entity Exposure Summary

| Entity | Exposed Operations | Notes |
|---|---|---|
| Asset | Create, Read, Update (draft), Delete (draft), Actions (allocate/transfer/returnAsset/retire/dispose/scheduleMaintenance) | Root, draft-enabled |
| AssetAllocation | Read only | System-managed |
| AssetMaintenance | Read, Update (limited), Action (completeMaintenance) | Technicians close out scheduled work |
| AssetAuditLog | Read only | System-managed, tamper-evident |
| AssetCategory | Read only | Reference/value help |
| InventorySummary | Read only, analytical | Dashboard aggregation |
| MaintenanceDue | Read only | Dashboard "due in 30 days" list |

## 4. Testing the Service

1. In Business Application Studio, right-click Service Binding → **Preview**.
2. Validate:
   - `GET /Asset` returns list with `StatusText` and `CurrentHolderName`.
   - `POST /Asset(...)/allocate` transitions status correctly and blocks when not In Stock.
   - `POST /Asset(...)/retire` on an Allocated asset force-closes the allocation and clears holder.
   - `GET /MaintenanceDue` returns only assets with an open, due-soon maintenance schedule.
3. Use Postman with IAS OAuth2 client credentials for automated regression testing (Phase 10).

## 5. Metadata Contract Notes

- `provider contract transactional_query` supports `$filter`, `$search`, `$orderby`, `$expand` for List Report/Object Page.
- Draft entity exposes `IsActiveEntity`/`HasActiveEntity`/`HasDraftEntity` automatically for Fiori Elements draft handling on asset registration.

---

**Next:** Phase 8 — SAP Fiori Elements.

---

# Phase 8 — SAP Fiori Elements
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Apps Delivered

| App | Floorplan | Primary Users |
|---|---|---|
| Asset Registration & Details | List Report + Object Page (draft) | IT Administrator, Asset Manager |
| Inventory Dashboard | Overview Page / Analytical List Page | Asset Manager, System Administrator |
| Maintenance Dashboard | List Report + Object Page | IT Administrator, Technician |
| My Assets | List Report (read-only, scoped) | Employee |

## 2. List Report — Asset

```json
{
  "sap.ui5": {
    "routing": {
      "targets": {
        "AssetList": {
          "type": "Component",
          "id": "AssetList",
          "name": "sap.fe.templates.ListReport",
          "options": {
            "settings": {
              "entitySet": "Asset",
              "variantManagement": "Page",
              "controlConfiguration": {
                "@com.sap.vocabularies.UI.v1.LineItem": {
                  "tableSettings": { "type": "ResponsiveTable" }
                }
              }
            }
          }
        },
        "AssetObjectPage": {
          "type": "Component",
          "id": "AssetObjectPage",
          "name": "sap.fe.templates.ObjectPage",
          "options": { "settings": { "entitySet": "Asset" } }
        }
      }
    }
  }
}
```

## 3. Object Page — Lifecycle Actions

Action buttons render conditionally based on `Status` and role, driven by `@UI.hidden` expressions bound to computed authorization fields:

| Button | Visible When |
|---|---|
| Allocate | `Status = 'IS'` |
| Transfer | `Status = 'AL'`, Asset Manager |
| Return | `Status = 'AL'` |
| Retire | `Status IN ('IS','AL')`, Asset Manager |
| Dispose | `Status = 'RE'`, Asset Manager |
| Schedule Maintenance | always (any status) |

## 4. Search, Filters, Value Helps

- **Search**: full-text on `AssetId`, `SerialNumber`, `Model`.
- **Filters**: `Status`, `CategoryId`, `CurrentEmployeeId` (department-scoped for Asset Managers), purchase date range.
- **Value Helps**: `CategoryId` bound to `Z_C_AssetCategory`; `EmployeeId` (for allocate/transfer) bound to `Z_C_Employee` filtered to active employees.

## 5. Maintenance Dashboard

Object Page facet showing scheduled vs. completed maintenance with a `completeMaintenance` action for technicians to log outcome and notes. `Z_C_MaintenanceDue` powers the "Due in 30 Days" table card on the Inventory Dashboard Overview Page.

## 6. Inventory Dashboard (Overview Page)

- KPI card: total assets by status (In Stock / Allocated / Retired)
- Chart card: assets by category (donut chart)
- Table card: assets due for maintenance in next 30 days, sourced from `Z_C_MaintenanceDue`

## 7. Navigation

```
Launchpad Tile: "Asset Registration" → List Report → Object Page (drilldown by AssetId)
Launchpad Tile: "Inventory Dashboard" → Overview Page → drill into filtered Asset List Report
Launchpad Tile: "My Assets" → List Report (filtered CurrentEmployeeId = current user, read-only)
```

## 8. Responsive Layout

Standard Fiori Elements responsive behavior — ResponsiveTable collapses to card view on mobile, Object Page sections stack vertically on narrow viewports, no custom CSS, fully Clean Core compliant.

---

**Next:** Phase 9 — Authorization.

---

# Phase 9 — Authorization
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. RBAC Model

| Business Role | Business Catalog | Apps | Scope |
|---|---|---|---|
| Z_BR_EMPLOYEE | Z_BC_ASSET_SELF_SERVICE | My Assets | Own allocated assets only, read-only |
| Z_BR_IT_ADMIN | Z_BC_ASSET_REGISTRATION | Asset Registration, Maintenance Dashboard | Register assets, schedule/complete maintenance |
| Z_BR_ASSET_MGR | Z_BC_ASSET_LIFECYCLE | Full Asset lifecycle actions | Allocate/transfer/retire/dispose, department-scoped or org-wide per config |
| Z_BR_SYS_ADMIN | Z_BC_ASSET_CONFIG | Configuration | Category setup, no transactional access |

## 2. Authorization Objects

```abap
"! Authorization object: Z_ASSET
"! Controls create/read/update/delete/action access to Asset
FIELDS:
  ACTVT    " Activity: 01 Create, 02 Change, 03 Display, 'ALLC','TRNS','RTN','RTR','DSP' (custom action activities)
  DEPT_ID  " Department scoping for Asset Manager
```

## 3. Instance Authorization in Behavior Implementation

```abap
CLASS lhc_Asset IMPLEMENTATION.

  METHOD get_instance_authorizations.
    DATA(lv_user) = cl_abap_context_info=>get_user_technical_name( ).

    LOOP AT keys INTO DATA(key).
      READ ENTITIES OF zi_asset IN LOCAL MODE
        ENTITY Asset FIELDS ( Status CurrentEmployeeId ) WITH VALUE #( ( %tky = key-%tky ) )
        RESULT DATA(assets).

      DATA(asset) = assets[ 1 ].
      DATA(lv_is_asset_mgr) = zcl_auth_util=>has_role( lv_user, 'Z_BR_ASSET_MGR' ).
      DATA(lv_is_it_admin)  = zcl_auth_util=>has_role( lv_user, 'Z_BR_IT_ADMIN' ).
      DATA(lv_is_holder)    = zcl_auth_util=>is_current_holder( lv_user, asset-CurrentEmployeeId ).

      APPEND VALUE #( %tky = key-%tky
                       %action-allocate  = COND #( WHEN lv_is_asset_mgr AND asset-Status = 'IS' THEN if_abap_behv=>auth-allowed ELSE if_abap_behv=>auth-unauthorized )
                       %action-transfer  = COND #( WHEN lv_is_asset_mgr AND asset-Status = 'AL' THEN if_abap_behv=>auth-allowed ELSE if_abap_behv=>auth-unauthorized )
                       %action-retire    = COND #( WHEN lv_is_asset_mgr THEN if_abap_behv=>auth-allowed ELSE if_abap_behv=>auth-unauthorized )
                       %action-dispose   = COND #( WHEN lv_is_asset_mgr AND asset-Status = 'RE' THEN if_abap_behv=>auth-allowed ELSE if_abap_behv=>auth-unauthorized )
                       %action-scheduleMaintenance = COND #( WHEN lv_is_it_admin THEN if_abap_behv=>auth-allowed ELSE if_abap_behv=>auth-unauthorized )
                     ) TO result.
    ENDLOOP.
  ENDMETHOD.

ENDCLASS.
```

The critical business rule (retired assets cannot be reallocated) is enforced **twice**: once as a validation (`validateStatusTransition`, Phase 6) and once implicitly through the `allocate` action's own precondition check — defense in depth against both direct OData manipulation and any future UI bug.

## 4. Business Roles → Role Mapping (BTP Cockpit)

| Business Role | Role Collection | IAS Group |
|---|---|---|
| Z_BR_EMPLOYEE | RC_ASSET_EMPLOYEE | All-Employees |
| Z_BR_IT_ADMIN | RC_ASSET_IT_ADMIN | IT-Support |
| Z_BR_ASSET_MGR | RC_ASSET_MANAGER | Asset-Managers |
| Z_BR_SYS_ADMIN | RC_ASSET_CONFIG | IT-Admins |

## 5. Security Validation Checklist

- [ ] Employee cannot see other employees' allocated assets via direct entity-key access.
- [ ] `allocate`/`transfer`/`retire`/`dispose` blocked server-side for non-Asset-Manager roles, even via direct OData batch call.
- [ ] Retired/Disposed assets hard-blocked from `allocate`/`transfer` at both validation and action-precondition layers.
- [ ] Maintenance completion restricted to IT Administrator/Technician role, not any authenticated user.
- [ ] Audit log entity confirmed read-only for all roles — no client can fabricate audit history.

---

**Next:** Phase 10 — Testing.

---

# Phase 10 — Testing
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Unit Tests (ABAP Unit + CDS Test Double Framework)

```abap
CLASS ltc_asset_fsm DEFINITION FOR TESTING DURATION SHORT RISK LEVEL HARMLESS.
  PRIVATE SECTION.
    METHODS:
      in_stock_to_allocated_is_valid FOR TESTING,
      retired_to_allocated_is_invalid FOR TESTING,
      disposed_is_terminal FOR TESTING.
ENDCLASS.

CLASS ltc_asset_fsm IMPLEMENTATION.

  METHOD in_stock_to_allocated_is_valid.
    DATA(lv_result) = zcl_asset_fsm=>is_valid_transition( iv_from = 'IS' iv_to = 'AL' ).
    cl_abap_unit_assert=>assert_true( lv_result ).
  ENDMETHOD.

  METHOD retired_to_allocated_is_invalid.
    DATA(lv_result) = zcl_asset_fsm=>is_valid_transition( iv_from = 'RE' iv_to = 'AL' ).
    cl_abap_unit_assert=>assert_false( lv_result ).
  ENDMETHOD.

  METHOD disposed_is_terminal.
    DATA(lv_result1) = zcl_asset_fsm=>is_valid_transition( iv_from = 'DI' iv_to = 'AL' ).
    DATA(lv_result2) = zcl_asset_fsm=>is_valid_transition( iv_from = 'DI' iv_to = 'IS' ).
    cl_abap_unit_assert=>assert_false( lv_result1 ).
    cl_abap_unit_assert=>assert_false( lv_result2 ).
  ENDMETHOD.

ENDCLASS.
```

### Behavior Test — Duplicate Serial Blocked

```abap
CLASS ltc_asset_validation DEFINITION FOR TESTING DURATION SHORT RISK LEVEL HARMLESS.
  PRIVATE SECTION.
    DATA environment TYPE REF TO if_cds_test_environment.
    METHODS: setup, teardown, duplicate_serial_raises_error FOR TESTING.
ENDCLASS.

CLASS ltc_asset_validation IMPLEMENTATION.
  METHOD setup.
    environment = cl_cds_test_environment=>create_for_multiple_cds(
                    i_for_entities = VALUE #( ( i_for_entity = 'ZI_Asset' ) ) ).
  ENDMETHOD.

  METHOD duplicate_serial_raises_error.
    environment->insert_test_data( i_data = VALUE zasset_tab(
      ( asset_uuid = '1' serial_number = 'SN-1001' status = 'IS' ) ) ).
    " Attempt to create a second asset with serial_number = 'SN-1001'
    " Assert failed-asset contains the new key with message DUPLICATE_SERIAL
  ENDMETHOD.

  METHOD teardown.
    environment->destroy( ).
  ENDMETHOD.
ENDCLASS.
```

## 2. Integration Tests

| Test | Steps | Expected |
|---|---|---|
| IT-01 Register → In Stock | Create draft asset, activate | Status = IS, audit log `REGISTERED` entry created |
| IT-02 Allocate | Allocate action on In Stock asset | Status = AL, `AssetAllocation` row created, `CurrentEmployeeId` set |
| IT-03 Illegal reallocation blocked | Retire an asset, then attempt allocate | Action fails with `RETIRED_ASSET_BLOCKED` |
| IT-04 Transfer | Transfer Allocated asset to new employee | Old allocation closed (`ReturnedAt` set), new allocation row created |
| IT-05 Retire force-closes allocation | Retire an Allocated asset directly | Allocation `ReturnedAt` auto-set, Status = RE, `CurrentEmployeeId` cleared |
| IT-06 Maintenance due dashboard | Schedule maintenance 10 days out | Asset appears in `MaintenanceDue` view; disappears once `PerformedDate` set |

## 3. Manual Test Cases

1. **TC-01**: IT Administrator registers an asset with a serial number already in use → UI blocks with clear duplicate-serial error.
2. **TC-02**: Asset Manager attempts to allocate a Retired asset directly via Object Page → Allocate button not visible; attempting via API returns unauthorized/blocked.
3. **TC-03**: Asset Manager transfers an asset between departments → audit trail shows both old and new holder with timestamps.
4. **TC-04**: Technician completes a scheduled maintenance with outcome "Flagged" → asset remains allocated, note visible in maintenance history.
5. **TC-05**: Employee views "My Assets" → sees only assets currently allocated to them, read-only.

## 4. Sample Test Data

| Asset ID | Category | Serial | Status | Holder |
|---|---|---|---|---|
| AST-0001 | LAPTOP | SN-1001 | Allocated | EMP001 |
| AST-0002 | MONITOR | SN-2001 | In Stock | — |
| AST-0003 | LAPTOP | SN-1002 | Retired | — |
| AST-0004 | PHONE | SN-3001 | Allocated | EMP004 (maintenance due in 5 days) |

## 5. Edge Cases

- Retiring an asset that has a maintenance record scheduled but not yet performed — maintenance history should remain visible for audit even after retirement.
- Two Asset Managers attempt to transfer the same asset simultaneously — ETag-based optimistic locking must reject the second write with a conflict message.
- Serial number entered with leading/trailing whitespace — normalization required before uniqueness check to avoid false negatives.
- Recurring maintenance (`RecurrenceDays > 0`) — completing one occurrence should auto-schedule the next; verify no duplicate due-list entries.

---

**Next:** Phase 11 — Deployment.

---

# Phase 11 — Deployment
## Project: Enterprise IT Asset Lifecycle Management

---

## 1. Prerequisites

- SAP BTP subaccount with **ABAP Environment (Steampunk)** entitlement
- SAP Business Application Studio dev space (ABAP Cloud Development type)
- SAP Cloud Identity Services configured for user authentication
- Git repository initialized and connected to Business Application Studio (abapGit)
- Master data available for initial load: Employees, Departments, Asset Categories

## 2. Deployment Steps

1. **Create ABAP Cloud Project** in Business Application Studio, connected to the target BTP ABAP Environment system.
2. **Import via abapGit**: pull `src/cds`, `src/behavior`, `src/classes`, `src/services` into customer development package `ZASSET_LIFECYCLE`.
3. **Activate objects in dependency order**:
   - Domains/Data Elements → Database Tables → Interface CDS → Projection CDS → Metadata Extensions → Behavior Definitions → Behavior Implementations → Service Definition → Service Binding.
4. **Load reference data**: `ZASSET_CATEGORY` (categories + default maintenance cycles), `ZDEPARTMENT`, `ZEMPLOYEE` (via CSV migration or integration with HR master data).
5. **Publish the Service Binding** and assign to Business Catalogs.
6. **Create/Assign Business Roles** (`Z_BR_EMPLOYEE`, `Z_BR_IT_ADMIN`, `Z_BR_ASSET_MGR`, `Z_BR_SYS_ADMIN`) to Role Collections, mapped to IAS groups.
7. **Deploy Fiori app tiles** to SAP Build Work Zone site, grouped by role-based launchpad groups.
8. **Run integration test suite** (Phase 10) against the deployed system before go-live sign-off.

## 3. Configuration Checklist

- [ ] Asset categories and default maintenance cycles seeded
- [ ] Employee/department master data loaded and validated
- [ ] Business roles assigned to correct IAS groups
- [ ] Unique index on `SERIAL_NUMBER` confirmed active in HANA
- [ ] Application Log object registered for lifecycle transition errors

## 4. Verification

- Smoke test: register → allocate → transfer → retire → dispose one asset end-to-end in QA.
- Confirm the illegal-transition guard blocks allocation of a retired asset in the deployed system (not just unit tests).
- Confirm Inventory Dashboard tiles render correct counts against seeded test data.
- Confirm Maintenance Dashboard "Due in 30 Days" list matches expected seeded schedule.

## 5. Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Duplicate serial not blocked | Unique index missing on `SERIAL_NUMBER` or validation not activated | Verify table index and re-check `validateUniqueSerial` activation |
| Allocate button never appears | Business role not assigned / role collection not mapped | Check BTP Cockpit role collection assignment |
| Retired asset still shows Allocate option | Metadata extension `@UI.hidden` expression cache stale | Clear Fiori metadata cache, redeploy |
| Maintenance Due list empty despite due assets | `Z_C_MaintenanceDue` where-clause date arithmetic mismatch with system timezone | Verify `$session.system_date` behavior in target landscape |

---

**Next:** Phase 12 — Documentation (see README.md).

---

# Enterprise IT Asset Lifecycle Management

Enterprise IT asset tracking application built on **SAP BTP ABAP Environment**, using **RAP**, **CDS Views**, **OData V4**, and **SAP Fiori Elements**. Governs the full asset lifecycle — registration, allocation, transfer, maintenance, return, retirement, and disposal — with a strictly enforced state machine, following **SAP Clean Core** principles.

## Contents

| Phase | Document |
|---|---|
| 1 | [Business Requirements](Phase1_Business_Requirements.md) |
| 2 | [Solution Architecture](Phase2_Solution_Architecture.md) |
| 3 | [Database Design](Phase3_Database_Design.md) |
| 4 | [CDS Views](Phase4_CDS_Views.md) |
| 5 | [Behavior Definitions](Phase5_Behavior_Definitions.md) |
| 6 | [Behavior Implementation](Phase6_Behavior_Implementation.md) |
| 7 | [Service Layer](Phase7_Service_Layer.md) |
| 8 | [SAP Fiori Elements](Phase8_SAP_Fiori_Elements.md) |
| 9 | [Authorization](Phase9_Authorization.md) |
| 10 | [Testing](Phase10_Testing.md) |
| 11 | [Deployment](Phase11_Deployment.md) |

## Architecture at a Glance

```
Fiori Elements Apps → OData V4 → RAP Behavior (Lifecycle FSM) → CDS Views → HANA
```

## Business Workflow

```
Register → Validate → Allocate → Employee Assignment → Maintenance Scheduling
        → Transfer (optional) → Return → Retire → Dispose
```

## Key Design Principles

- **Clean Core**: no modifications to SAP standard objects; all extensions in customer namespace `Z*`.
- **Strict lifecycle state machine**: enforced in behavior validations — the core safeguard ensuring retired assets can never be reallocated (BR-07 from Phase 1).
- **Tamper-evident audit log**: `AssetAuditLog` is system-write-only; no client can create or alter audit history.
- **Server-side authorization**: every lifecycle action is authorization-checked in the behavior implementation, not only hidden in the UI.

## Roles

| Role | Capability |
|---|---|
| Employee | View own allocated assets (read-only) |
| IT Administrator | Register assets, schedule/complete maintenance |
| Asset Manager | Allocate, transfer, retire, dispose assets |
| System Administrator | Category/configuration management |

## Getting Started (Developer)

1. Read Phase 1–2 for business and architecture context.
2. Clone this repo into SAP Business Application Studio.
3. Follow [Phase 11 Deployment](Phase11_Deployment.md) to activate objects in order.
4. Load reference data (categories, departments, employees) before functional testing.
5. Run the ABAP Unit test classes described in [Phase 10](Phase10_Testing.md).

## API Documentation

Service metadata is self-describing via the OData V4 `$metadata` endpoint once deployed:
`/sap/opu/odata4/sap/z_ui_asset_lifecycle_o4/srvd/sap/z_ui_asset_lifecycle_o4/0001/$metadata`

See [Phase 7](Phase7_Service_Layer.md) for entity/action reference.

## User Guide (Summary)

- **IT Administrators**: Launchpad → "Asset Registration" → New → enter serial/category/model → Activate.
- **Asset Managers**: open an asset's Object Page → Allocate/Transfer/Retire/Dispose based on current status.
- **Employees**: Launchpad → "My Assets" to see current holdings, read-only.
- **Technicians**: Maintenance Dashboard → open due item → log outcome via "Complete Maintenance".

## Technical Documentation

Full technical detail — data model, CDS annotations, lifecycle state machine, behavior logic, authorization design, test strategy, and deployment runbook — is captured across Phases 1–11 above.

## Future Enhancements (Designed, Not Yet Implemented)

The following are architected for but intentionally excluded from the current implementation scope, per Phase 1 requirements:

| Enhancement | Notes |
|---|---|
| QR Code Asset Tracking | Would extend `Asset` with a `QRCodeValue` field and a scan-to-lookup Fiori action |
| Barcode Integration | Similar pattern to QR; requires device camera API integration in a custom UI extension |
| SAP Build Process Automation | Could automate multi-step transfer approvals for high-value assets |
| SAP Mobile Start Integration | Push notifications for maintenance due / pending returns |
| Email Notifications | Triggered from lifecycle determinations via SAP BTP email service |
| SAP Analytics Cloud Dashboard | `Z_C_InventorySummary` is already analytics-query-enabled, ready for SAC consumption |
| AI-Based Maintenance Recommendations | Would follow the same interface-driven AI pattern used in Project 2 (`ZIF_AI_CAPACITY_SERVICE` analog) |
| Predictive Maintenance (SAP AI Core) | Requires historical failure data collection first; current `AssetMaintenance` table is schema-ready |
| Vendor Performance Analytics | Would require a new `ZVENDOR` master table and linkage from `Asset.PURCHASE` data |

These are explicitly **not implemented** in the current codebase to keep the delivered scope clean-core and production-focused; the data model and layering were designed so each can be added without breaking existing consumers.

---
*Project 3 of 3 — Enterprise SAP BTP Portfolio*
