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
