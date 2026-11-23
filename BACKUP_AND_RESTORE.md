# BACKUP & RESTORE PROCEDURE — DSS-PIK PRODUCTION

## 1. Scheduled Daily Database Backup (Ubuntu `cron`)
```bash
# Add to root crontab (runs daily at 02:00 WIB; uses MYSQL_PWD to avoid exposing password in ps):
0 2 * * * docker exec dss_pik_prod_mariadb bash -c 'MYSQL_PWD="$MARIADB_PASSWORD" mariadb-dump --single-transaction --routines --triggers -u"$MARIADB_USER" "$MARIADB_DATABASE"' | gzip > /srv/dss-pik-backups/cews_db_$(date +\%Y\%m\%d_\%H\%M\%S).sql.gz
```

## 2. Manual On-Demand Backup (Before Any Upgrade)
```bash
docker exec dss_pik_prod_mariadb bash -c 'MYSQL_PWD="$MARIADB_PASSWORD" mariadb-dump --single-transaction --routines --triggers -u"$MARIADB_USER" "$MARIADB_DATABASE"' | gzip > /srv/dss-pik-backups/cews_db_pre_upgrade_$(date +%Y%m%d_%H%M%S).sql.gz
```

## 3. Restore from Backup
```bash
# Always back up current state first!
gunzip -c /srv/dss-pik-backups/<backup_file>.sql.gz | docker exec -i dss_pik_prod_mariadb bash -c 'MYSQL_PWD="$MARIADB_ROOT_PASSWORD" mariadb -u root "$MARIADB_DATABASE"'
```

> **WARNING**: Never run `docker compose down -v`. Named volume `dss_pik_prod_mariadb_data` holds the live operational database.
