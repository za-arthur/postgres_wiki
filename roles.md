# Check role membership

```sql
SELECT
    r.rolname       AS member_role,
    m.rolname       AS target_role,
    am.admin_option AS has_admin_option
FROM pg_auth_members am
JOIN pg_roles r ON r.oid = am.member
JOIN pg_roles m ON m.oid = am.roleid
WHERE r.rolname = 'postgres';
```

```sql
WITH RECURSIVE x AS
(
  SELECT grantor::regrole AS granted_by,
         member::regrole,
         roleid::regrole AS role,
         member::regrole || ' -> ' || roleid::regrole AS path
  FROM pg_auth_members AS m
  UNION ALL
  SELECT m.grantor::regrole,
         x.member::regrole,
         m.roleid::regrole,
         x.path || ' -> ' || m.roleid::regrole
  FROM pg_auth_members AS m
    JOIN x ON m.member = x.role
  )
SELECT granted_by, member, role, path
FROM x
ORDER BY member::text, role::text;
```

# Check role permissions

Check if a role has explicit permissions on a table

```sql
SELECT 
    r.rolname,
    CASE WHEN has_table_privilege('<role_name>', '<table_name>', 'SELECT') 
         THEN 'YES' ELSE 'NO' END as can_select
FROM pg_roles r 
WHERE r.rolname = '<role_name>';
```

Check explicit permissions on a schema

```sql
SELECT 
    r.rolname,
    n.nspname,
    CASE WHEN has_schema_privilege(r.rolname, n.nspname, 'USAGE') 
         THEN 'YES' ELSE 'NO' END as has_usage
FROM pg_roles r
CROSS JOIN pg_namespace n
WHERE r.rolname = '<role_name>'
  AND n.nspname = '<nspname>';
```

# Default privileges

```sql
SELECT
    pg_get_userbyid(d.defaclrole) AS role,
    n.nspname AS schema,
    d.defaclobjtype AS object_type,
    d.defaclacl
FROM pg_default_acl d
LEFT JOIN pg_namespace n ON n.oid = d.defaclnamespace
ORDER BY role, schema, object_type;
```
