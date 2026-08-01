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
