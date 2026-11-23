# UBUNTU SERVER INSTALLATION GUIDE — DSS-PIK `v1.0.0`

## 1. Server Prerequisites (Confirm with BMKG IT)
- **OS**: Ubuntu Server 22.04 LTS or 24.04 LTS (`x86_64` / `amd64`)
- **Resources**: >= 4 vCPU, >= 8 GB RAM, >= 50 GB free disk space (`/srv/dss-pik` + `/srv/dss-pik-backups`)
- **Runtime**: Docker Engine >= 24.x and Docker Compose v2 plugin (`docker compose version`)
- **Offline Ready**: No external internet access is required once this release package is transferred.

## 2. Prepare Host Directories & Transfer Package
```bash
sudo mkdir -p /srv/dss-pik /srv/dss-pik-backups
sudo chown -R $USER:$USER /srv/dss-pik /srv/dss-pik-backups
# Copy DSS_PIK_PRODUCTION_RELEASE to /srv/dss-pik/release/
cd /srv/dss-pik/release
sha256sum -c 06_VERIFICATION/SHA256SUMS.txt
```

## 3. Load Offline Docker Images
```bash
gunzip -c 01_DOCKER_IMAGES/dss-pik_1.0.0_linux_amd64.tar.gz | sudo docker load
gunzip -c 01_DOCKER_IMAGES/mariadb_11.4_linux_amd64.tar.gz | sudo docker load
sudo docker images | grep -E "dss-pik|mariadb"
```

## 4. Stage Compose & Operational Data
```bash
cp 02_COMPOSE/compose.production.yaml /srv/dss-pik/
cp 02_COMPOSE/.env.production.example /srv/dss-pik/.env.production
chmod 600 /srv/dss-pik/.env.production
cp -r 04_OPERATIONAL_DATA/data /srv/dss-pik/data
```

## 5. Provision Unique Production Credentials
Edit `/srv/dss-pik/.env.production` and generate unique strong credentials on the server:
```bash
openssl rand -hex 32          # -> DSS_PIK_SESSION_SECRET
openssl rand -base64 24       # -> CEWS_DB_PASSWORD & MARIADB_ROOT_PASSWORD (use two distinct values)
sudo docker run --rm -it dss-pik:1.0.0 python -c "import bcrypt, getpass; print(bcrypt.hashpw(getpass.getpass('Operator password: ').encode(), bcrypt.gensalt()).decode())"
# Wrap the bcrypt hash in single quotes '...' in .env.production!
```

## 6. Initialize MariaDB & Restore Operational Database (First Install Only)
```bash
cd /srv/dss-pik
sudo docker compose -f compose.production.yaml --env-file .env.production up -d mariadb
# Wait until mariadb is healthy:
sudo docker compose -f compose.production.yaml ps

# Guarded first-time restore (uses MYSQL_PWD env so password never appears in process args):
sudo docker exec -i dss_pik_prod_mariadb bash -c '
  export MYSQL_PWD="$MARIADB_ROOT_PASSWORD"
  cnt=$(mariadb -u root -N -B "$MARIADB_DATABASE" -e "SHOW TABLES LIKE '\''stations'\'';" | wc -l)
  if [ "$cnt" -gt 0 ]; then echo "ABORT: Database already populated!"; exit 1; fi
  mariadb -u root "$MARIADB_DATABASE"
' < /srv/dss-pik/release/03_DATABASE/RESTRICTED_cews_db_release_20261123.sql
```

## 7. Start DSS-PIK Application & Verify
```bash
cd /srv/dss-pik
sudo docker compose -f compose.production.yaml --env-file .env.production up -d
sudo docker compose -f compose.production.yaml ps
curl -fsS http://127.0.0.1:8000/health
```

## 8. Daily Operations
- **Status**: `sudo docker compose -f compose.production.yaml ps`
- **Logs**: `sudo docker compose -f compose.production.yaml logs -f --tail=100 app`
- **Safe Stop**: `sudo docker compose -f compose.production.yaml stop` (NEVER use `down -v`!)
- **Restart**: `sudo docker compose -f compose.production.yaml restart`
