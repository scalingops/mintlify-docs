### What this is

Operator runbook to migrate all RLS policies to the enhanced `can()`/`can_with_company()` pattern.

### Prereqs
- Confirm helpers exist: `can`, `can_with_company`, `has_permission`, `check_tenant_access`, `set_tenant_id_from_user`
- Confirm `v_user_effective_permissions` has `security_invoker = true`

### Steps
1) Review mapping at `scripts/rls/table_mapping.json`. Adjust resources or keys if needed.
2) Apply migration: `supabase/migrations/20250914_migrate_rls_to_can.sql`.
3) Validate: run `scripts/rls/validate_rls.sql` and check PASS/FAIL.
4) Inspect policy diff: `select * from public._rls_policy_diff order by tablename, status`.
5) Re-run ANALYZE-intensive workloads and check EXPLAIN on representative queries.

### Rollback (per table)
- Restore policies from `rls_policy_snapshots` for a table:
```sql
-- Example rollback template
DO $$
DECLARE r record;
BEGIN
  PERFORM public._drop_all_policies('public','your_table');
  FOR r IN
    SELECT *
    FROM public.rls_policy_snapshots
    WHERE schemaname='public' and tablename='your_table'
      AND captured_at = (select max(captured_at) from public.rls_policy_snapshots)
  LOOP
    EXECUTE format('CREATE POLICY %I ON %I.%I FOR %s USING (%s) %s',
      r.policyname, r.schemaname, r.tablename, r.cmd,
      coalesce(r.qual, 'true'),
      CASE WHEN r.with_check IS NOT NULL THEN format('WITH CHECK (%s)', r.with_check) ELSE '' END
    );
  END LOOP;
END$$;
```

### Acceptance checks
- Each table has exactly 4 canonical policies calling `can()` or `can_with_company()`.
- Global tables: SELECT allows `is_global = true OR can('resource','read',tenant_id)`.
- Tenant triggers present on insert for tenant-scoped tables.
- Indexes present as per `docs/RLS_SECURITY_GUIDE.md`.


