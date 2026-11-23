# UPDATE & ROLLBACK PROCEDURE — DSS-PIK PRODUCTION

## 1. Application Image Upgrade (Zero Data Loss)
1. Take a pre-upgrade MariaDB backup (see `BACKUP_AND_RESTORE.md`).
2. Load the new versioned image archive:
   ```bash
   gunzip -c dss-pik_<NEW_VERSION>_linux_amd64.tar.gz | sudo docker load
   ```
3. Update `DSS_PIK_VERSION=<NEW_VERSION>` in `/srv/dss-pik/.env.production`.
4. Recreate **only** the `app` container (MariaDB & persistent volumes remain untouched):
   ```bash
   cd /srv/dss-pik
   sudo docker compose -f compose.production.yaml --env-file .env.production up -d --no-deps app
   ```
5. Verify `curl -fsS http://127.0.0.1:8000/health` returns HTTP 200.

## 2. Instant Rollback
1. Set `DSS_PIK_VERSION=<PREVIOUS_VERIFIED_VERSION>` in `/srv/dss-pik/.env.production`.
2. Run:
   ```bash
   sudo docker compose -f compose.production.yaml --env-file .env.production up -d --no-deps app
   ```
3. If a database schema migration was executed during the upgrade, restore the pre-upgrade SQL dump.

> **Application Image Rebuild vs. Schema Migration**: Replacing `dss-pik:<ver>` only replaces stateless application code. Database schema migrations are separate, controlled operations requiring pre-migration backups and staging validation.
