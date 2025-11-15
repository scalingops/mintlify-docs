# RPC Performance Audit

## Executive Summary

**Audit Date:** 2025-10-26
**Total RPCs Analyzed:** 281 functions
**RPCs Using SELECT *:** 20 functions (7%)
**Status:** 🔴 **Critical Issues Found**

### Key Findings

- **3 high-impact RPCs** are returning excessive data (21-36 columns) via SELECT *
- `get_current_user_profile` is called **155+ times** across the codebase
- Estimated overhead: **~15-30 unnecessary columns per call** × 155 calls = **massive over-fetching**
- Several pagination RPCs properly use explicit columns ✅

---

## Critical RPCs Using SELECT *

### 1. 🔴 `get_current_user_profile` - **CRITICAL**

**Impact:** Called 155+ times across codebase
**Current Behavior:** Returns ALL 21 columns from `users` table
**Actual Usage:** Only ~5 columns actually used (`tenant_id`, `user_id`, `company_id`, `is_primary_admin`, `role_id`)

**Column Breakdown:**
```sql
-- ALL 21 COLUMNS RETURNED:
email, full_name, avatar_url, created_at, updated_at, user_id, role_id,
is_primary_admin, status, tenant_id, last_login, invitation_sent_at,
invitation_expires_at, invitation_token, company_id, user_type, imported_at,
imported_by, integration_user_id, id, version

-- ACTUALLY NEEDED (~5 columns):
user_id, tenant_id, company_id, is_primary_admin, role_id
```

**Current Definition:**
```sql
CREATE OR REPLACE FUNCTION public.get_current_user_profile()
 RETURNS SETOF users
 LANGUAGE sql SECURITY DEFINER SET search_path TO ''
AS $function$
  SELECT * FROM public.users WHERE user_id = auth.uid();
$function$
```

**Estimated Savings:** 76% reduction in data transfer (16/21 columns eliminated)

**Usage Examples:**
- `lib/permissions/check-page-access.ts`: Uses only `is_primary_admin`
- `components/home/TeamWidgetServer.tsx`: Uses only `company_id`
- `app/training/learning-paths/actions.ts`: Called 24 times in one file

**Recommendation:** Create explicit column version returning only essential fields

---

### 2. 🔴 `get_planner_items_for_report` - **HIGH IMPACT**

**Impact:** Returns full report data for exports
**Current Behavior:** Returns ALL 36 columns from `v_planner_items_detailed` view
**Actual Usage:** Used in `app/actions/planner/getPlannerItemsForReport.ts`

**Column Breakdown:**
```sql
-- ALL 36 COLUMNS RETURNED:
tenant_id, company_id, company_name, planner_item_id, title, description,
item_type, status, business_value, hours, cost_estimate, start_date,
due_date, end_date, public_notes, private_notes, is_archived, created_by,
created_at, updated_at, is_overdue, fiscal_year, fiscal_q,
linked_goals_count, linked_compliance_count, linked_meetings_count,
linked_projects_count, tag_count, goals_json, compliance_json,
meetings_json, projects_json, opportunities_json, tags_json,
linked_secure_score_count, secure_score_json
```

**Current Definition:**
```sql
CREATE OR REPLACE FUNCTION public.get_planner_items_for_report(...)
 RETURNS SETOF v_planner_items_detailed
 LANGUAGE sql STABLE SECURITY DEFINER
AS $function$
SELECT * FROM v_planner_items_detailed
WHERE tenant_id = p_tenant_id
  AND (p_company_id IS NULL OR company_id = p_company_id)
  AND (p_date_from IS NULL OR due_date >= p_date_from)
  AND (p_date_to IS NULL OR due_date <= p_date_to)
  AND (p_types IS NULL OR item_type = ANY (p_types))
  AND (p_statuses IS NULL OR status = ANY (p_statuses))
  AND (p_include_archived OR NOT is_archived);
$function$
```

**Recommendation:**
- Analyze actual report column requirements
- May need all columns for reporting, but should verify
- Consider creating slimmer version for previews vs full exports

---

### 3. 🟡 `get_user_by_auth_id` - **MODERATE**

**Impact:** Returns ALL 21 columns from `users` table
**Current Behavior:** Same as `get_current_user_profile`
**Usage:** Only found in type files (may be unused or lightly used)

**Current Definition:**
```sql
CREATE OR REPLACE FUNCTION public.get_user_by_auth_id(auth_uid uuid)
 RETURNS SETOF users
 LANGUAGE sql SECURITY DEFINER SET search_path TO ''
AS $function$
  SELECT * FROM public.users WHERE user_id = auth_uid;
$function$
```

**Recommendation:**
- Audit actual usage in application
- If used, create explicit column version
- If unused, consider deprecating

---

## Pagination RPCs - ✅ GOOD EXAMPLES

The following pagination RPCs use **explicit column lists** and are good examples:

### ✅ `fetch_companies_page_v1`
- Returns only 13 specific columns
- Uses proper pagination with offset/limit
- Includes total_count for UI

### ✅ `fetch_compliance_checks_page_v2`
- Returns only 12 specific columns
- Uses proper pagination
- Includes filtering capabilities

### ✅ `fetch_devices_page_v3`
- Returns only 11 specific columns (removed device_metadata in v3)
- Uses proper pagination
- Good example of versioning to reduce columns

### ✅ `fetch_goals_page_v2`
- Returns only 15 specific columns
- Proper pagination with aggregates
- Clean separation of concerns

### ⚠️ `fetch_compliance_checks_keyset`
- Uses **keyset pagination** (cursor-based) ✅
- Returns only 12 specific columns ✅
- **Recommendation:** Migrate offset-based RPCs to keyset pattern for better performance

---

## Other RPCs Using SELECT *

### 🟢 Lower Priority (Internal/Triggers)

4. `verify_api_key` - API auth (minimal impact, small table)
5. `import_company_users` - Bulk operations (infrequent)
6. `copy_template_survey_to_tenant` - Template operations (infrequent)
7. `handle_new_user` - Trigger function (acceptable for triggers)
8. `process_nps_auto_actions` - Trigger function (acceptable for triggers)
9. `can_row` - Internal auth check
10. `copy_default_tenant_settings` - Setup function (one-time)
11. `current_user_can_access_company` - Auth check
12. `evaluate_survey_group_rules` - Survey logic
13. `get_user_accessible_companies` - Returns only company_id (TABLE return type)
14. `get_user_context` - Small return set (5 columns in TABLE definition)
15. `log_ticket_access` - Logging (void return)
16. `send_pending_nps_surveys` - Background job (void return)
17. `sync_user_metadata_by_email` - Sync operation
18. `test_can_access_debug` - Debug function
19. `rpc_template_upsert` - Template management
20. `pgmq_read` - External library (cannot modify)

---

## Performance Profiling

### Test Methodology

Need to run actual profiling queries to measure:
1. Execution time
2. Row count returned
3. Data volume (bytes)
4. Query plan analysis

### Sample Profiling Query Template

```sql
-- Profile get_current_user_profile
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT * FROM public.users WHERE user_id = '[test-uuid]';

-- Measure data volume
SELECT
  pg_column_size(row(*)) as row_size_bytes,
  pg_column_size(row(user_id, tenant_id, company_id, is_primary_admin, role_id)) as optimized_size_bytes
FROM public.users
WHERE user_id = '[test-uuid]';
```

---

## Recommendations

### Immediate Actions (High Priority)

1. **Refactor `get_current_user_profile`** (CRITICAL)
   - Create new version with explicit 5 columns
   - Estimated impact: 155+ call sites × 76% data reduction
   - Migration path: Update AppDataProvider and core utilities

2. **Audit `get_planner_items_for_report`** (HIGH)
   - Verify all 36 columns are actually needed for reports
   - Consider creating two versions: full export vs preview

3. **Investigate `get_user_by_auth_id`** (MEDIUM)
   - Search actual usage beyond type files
   - Refactor or deprecate based on findings

### Medium Priority

4. **Implement keyset pagination pattern across all list RPCs**
   - `fetch_compliance_checks_keyset` is good example
   - Better performance for large datasets
   - Migrate offset-based RPCs:
     - `fetch_companies_page_v1`
     - `fetch_company_users_page_v1`
     - `fetch_devices_page_v3`
     - `fetch_goals_page_v2`
     - `fetch_tenant_users_page_v1`

5. **Review lower-priority SELECT * functions**
   - Most are triggers/internal functions (acceptable)
   - `verify_api_key` should be audited for auth performance

### Testing Requirements

1. **Unit Tests** - Assert explicit column lists
   ```typescript
   // Example test
   test('get_current_user_profile returns only essential columns', async () => {
     const profile = await getCurrentUserProfile();
     const keys = Object.keys(profile);
     expect(keys).toEqual(['user_id', 'tenant_id', 'company_id', 'is_primary_admin', 'role_id']);
   });
   ```

2. **Performance Tests** - Measure before/after
3. **Integration Tests** - Ensure no breaking changes

---

## Migration Strategy

### Phase 1: Immediate Fixes (Week 1)

1. Create `get_current_user_profile_v2` with explicit columns
2. Create migration helper/wrapper
3. Update core utilities (AppDataProvider, check-page-access)
4. Test thoroughly in staging

### Phase 2: Rollout (Week 2)

1. Update remaining 150+ call sites
2. Deprecate old function
3. Monitor performance improvements

### Phase 3: Additional Optimizations (Week 3-4)

1. Audit and fix other SELECT * RPCs
2. Implement keyset pagination pattern
3. Create performance benchmarks

---

## Acceptance Criteria

- [ ] No RPC uses SELECT * (except acceptable triggers)
- [ ] `get_current_user_profile` returns only 5 essential columns
- [ ] `get_planner_items_for_report` audited and optimized
- [ ] All pagination RPCs use keyset pagination
- [ ] Unit tests verify explicit column lists
- [ ] Performance metrics show improvement
- [ ] Documentation updated

---

## Appendix: Full RPC List

### All 281 Public Functions

See database query results for complete list. Key categories:

- **Pagination RPCs:** 10 functions (mostly good)
- **Auth/Permission RPCs:** 25+ functions
- **Integration RPCs:** 50+ functions
- **Trigger Functions:** 40+ functions
- **Utility Functions:** 100+ functions
- **Business Logic RPCs:** 50+ functions

### Functions by Pattern

- **Explicit columns (GOOD):** 261 functions (93%)
- **SELECT * (NEEDS FIX):** 20 functions (7%)
- **SETOF table (subset of SELECT *):** 4 functions (highest priority)

---

**Next Steps:**

1. Run performance profiling on critical RPCs
2. Create refactoring PRs for `get_current_user_profile`
3. Implement unit tests for column lists
4. Measure and document performance improvements
