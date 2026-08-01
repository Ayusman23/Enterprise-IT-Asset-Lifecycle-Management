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
