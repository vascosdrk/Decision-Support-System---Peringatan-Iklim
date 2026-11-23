# DSS-PIK `v1.0.0` — BMKG IT TECHNICIAN DELIVERY README

**Project**: Decision Support System Peringatan Iklim (DSS-PIK) Kepulauan Riau  
**Institution**: BMKG Stasiun Meteorologi Kelas II Raja Haji Fisabilillah Tanjungpinang  
**Release Version**: `v1.0.0` (`dss-pik:1.0.0` + `mariadb:11.4`)  
**Target Architecture**: Ubuntu Server 22.04 / 24.04 LTS (`linux/amd64`, `x86_64`)  
**Delivery Status**: **`READY FOR TECHNICIAN CONFIGURATION AND SERVER ACCEPTANCE`**

---

## 1. Exact Deliverables to Hand Over (`F:\TNJ\paper\DSS_PIK_DELIVERY_v1.0.0\`)

| Deliverable | Size | Files | SHA-256 | Transfer Channel |
|---|---:|---:|---|---|
| **`DSS-PIK_v1.0.0_TECHNICIAN_DEPLOYMENT.zip`** | `530,557,765` bytes (`506 MB`) | 319 | `379f16aca2e6a977f410edafcb807a4428125eea731ac510e15cbf67acb7dd70` | Standard internal media / transfer |
| **`DSS-PIK_v1.0.0_RESTRICTED_DATABASE/RESTRICTED_cews_db_release_20261123.sql`** | `41,682,261` bytes (`39.8 MB`) | 1 (+ checksum & README) | `fcbdca6c539f7b73ef5190a86aaa41a003e13beb96981019fd5140dcd1446d45` | **Separate institution-approved encrypted storage/transfer** |

*(Zero development passwords, `.env`, or encryption keys are included in either deliverable.)*

---

## 2. Initial Deployment Steps (First-Time Installation)

1. Verify `DSS-PIK_v1.0.0_TECHNICIAN_DEPLOYMENT.zip` SHA-256 on the Ubuntu Server and extract to `/srv/dss-pik/release/`.
2. Run `cd /srv/dss-pik/release/DSS-PIK_v1.0.0 && sha256sum -c 06_VERIFICATION/SHA256SUMS.txt`.
3. Load offline Docker images:
   - `gunzip -c 01_DOCKER_IMAGES/dss-pik_1.0.0_linux_amd64.tar.gz | sudo docker load`
   - `gunzip -c 01_DOCKER_IMAGES/mariadb_11.4_linux_amd64.tar.gz | sudo docker load`
4. Copy `compose.production.yaml`, `.env.production.example` -> `.env.production` (`chmod 600`), and `04_OPERATIONAL_DATA/data/` -> `/srv/dss-pik/data/`.
5. Generate unique production credentials on the server (`CEWS_DB_PASSWORD`, `MARIADB_ROOT_PASSWORD`, `DSS_PIK_SESSION_SECRET`, single-quoted bcrypt `DSS_PIK_PASSWORD_HASH`).
6. Start MariaDB (`up -d mariadb`), verify `RESTRICTED_DB_SHA256SUMS.txt`, and run the guarded first-time SQL restore using `MYSQL_PWD="$MARIADB_ROOT_PASSWORD"` (per `05_TECHNICIAN_GUIDES/UBUNTU_INSTALLATION.md`).
7. Start application (`up -d`), verify `curl -fsS http://127.0.0.1:8000/health` returns HTTP 200, and configure reverse proxy (HTTPS/TLS) + `ufw` firewall per `05_TECHNICIAN_GUIDES/NETWORK_AND_SECURITY.md`.

---

## 3. First-Install vs. Subsequent Updates

- **First-Time Install**: Initializes persistent volume `dss_pik_prod_mariadb_data` and imports `RESTRICTED_cews_db_release_20261123.sql` once.
- **Subsequent App Updates**: NEVER run `docker compose down -v` and NEVER re-import the initial SQL dump! Back up MariaDB first, load the new `dss-pik:<ver>` image, update `DSS_PIK_VERSION` in `.env.production`, and run `docker compose -f compose.production.yaml --env-file .env.production up -d --no-deps app`.

---

## 4. Information Required from BMKG IT Technician & Remaining Server Tests

- Official Ubuntu Server IP address & DNS hostname
- Authorized office LAN client subnet(s)
- HTTPS/TLS certificate & reverse proxy (Nginx/Caddy) configuration
- Ubuntu `ufw` firewall rules & daily backup `cron` destination
- **Remaining Acceptance Tests (on Office Server)**: Multi-PC office LAN browser access, HTTPS/TLS validation, and unauthorized-client rejection from outside the approved subnet.
