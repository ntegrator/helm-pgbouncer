# PgBouncer Helm Chart 
> based on Bitnami docker image 

PgBouncer - lightweight connection pooler for PostgreSQL.  
Homepage: https://www.pgbouncer.org/  

## TL;DR
```bash
git clone https://github.com/ntegrator/helm-pgbouncer.git
helm install my-release ./helm-pgbouncer
```

## Example of usage

### 1. Preparing your DB for using PgBouncer
[Please read the PgBouncer documentation first](https://www.pgbouncer.org/config.html)

Execute on your DB Master node:
```sql
\c your_app_db

CREATE USER pgbouncer WITH PASSWORD 'superpassword';

CREATE SCHEMA pgbouncer AUTHORIZATION pgbouncer;

GRANT ALL ON SCHEMA pgbouncer TO pgbouncer;

CREATE OR REPLACE FUNCTION pgbouncer.user_lookup(in i_username text, out uname text, out phash text)
RETURNS record AS $$
BEGIN
    SELECT usename, passwd FROM pg_catalog.pg_shadow
    WHERE usename = i_username INTO uname, phash;
    RETURN;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

REVOKE ALL ON FUNCTION pgbouncer.user_lookup(text) FROM public, pgbouncer;

GRANT EXECUTE ON FUNCTION pgbouncer.user_lookup(text) TO pgbouncer;
```

### 2. Set `PGBOUNCER_AUTH_QUERY` in PgBouncer Helm chart:
```yaml
env:
...
  PGBOUNCER_AUTH_QUERY: "SELECT * FROM pgbouncer.user_lookup($1)"
  ...
```

### 3. Set other variables in PgBouncer Helm chart if needed
```bash
vim helm-pgbouncer/values.yaml
```

### 4. Install Helm release
```bash
helm install my-release ./helm-pgbouncer
```

### 5. Change postgres connection string in your application.  
Just replace `...postgresql_host:postgresql_port/...` on `...pgbouncer_host:pgbouncer_port/...?binary_parameters=yes`  
Example:  
> before - `app_user:mega_password@1.2.3.4:5432/your_app_db`  
> after  - `app_user:mega_password@pgbouncer:6432/your_app_db?binary_parameters=yes`
