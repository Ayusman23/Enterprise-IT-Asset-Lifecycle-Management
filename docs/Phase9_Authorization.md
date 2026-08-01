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
