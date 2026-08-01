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
