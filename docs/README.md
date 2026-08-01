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
