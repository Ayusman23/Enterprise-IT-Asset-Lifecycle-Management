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
