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
