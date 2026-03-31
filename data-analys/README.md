# Installation:

on okd, you have to apply scc permisions. To install it execute the following:

```bash
oc adm policy add-scc-to-group anyuid system:serviceaccounts:data-analys
oc adm policy add-scc-to-group privileged system:serviceaccounts:data-analys

# confirm SA is using sts (if no response, is using default)
oc -n data-analys get sts airbyte-db -o jsonpath='{.spec.template.spec.serviceAccountName}{"\n"}'

# in case of default
oc auth can-i use scc/privileged --as=system:serviceaccount:data-analys:default -n data-analys

# force recreate
oc -n data-analys delete pod airbyte-db-0 --ignore-not-found
```

## Metabase:

Before deploy metabase, you have to apply this sql on phpmyadmin:

```bash
CREATE DATABASE IF NOT EXISTS metabase;
CREATE USER IF NOT EXISTS 'metabase'@'%' IDENTIFIED BY 'change-me-metabase-db-pass';
GRANT ALL PRIVILEGES ON metabase.* TO 'metabase'@'%';
FLUSH PRIVILEGES;

ALTER USER 'metabase'@'%' IDENTIFIED WITH mysql_native_password BY 'change-me-metabase-db-pass';
FLUSH PRIVILEGES;
```

## External Secrets (Vault)

Secrets are managed with `ExternalSecret` resources (no direct `Secret` manifests).

Required Vault paths and properties:

- `apps/data-analys/mysql`
  - `MYSQL_ROOT_PASSWORD`
  - `MYSQL_USER`
  - `MYSQL_PASSWORD`
- `apps/data-analys/metabase`
  - `MB_DB_USER`
  - `MB_DB_PASS`
