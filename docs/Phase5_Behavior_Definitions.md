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
