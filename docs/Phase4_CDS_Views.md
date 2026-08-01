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
