# NETWORK, FIREWALL & SECURITY CONFIGURATION (UBUNTU SERVER)

## 1. Mandatory Security Defaults
- `CEWS_DOCKER_LOCAL_ONLY` is hardcoded to `"false"` in `compose.production.yaml`. Never enable it in production.
- MariaDB (`dss_pik_prod_mariadb`) is strictly internal (`dss_pik_prod_internal` bridge); port `3306` is **never** published to the host or office LAN.
- Application container defaults to `127.0.0.1:8000:8000` behind an approved reverse proxy.

## 2. Reverse Proxy & HTTPS/TLS (Nginx Example)
Install Nginx on Ubuntu Server and terminate TLS (internal BMKG CA or approved cert) on port `443`, proxying to `http://127.0.0.1:8000`:
```nginx
server {
    listen 443 ssl http2;
    server_name dss-pik.bmkg.local; # Replace with official BMKG hostname/IP
    ssl_certificate     /etc/ssl/certs/dss-pik.crt;
    ssl_certificate_key /etc/ssl/private/dss-pik.key;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 3. Ubuntu Firewall (`ufw`) Policy
```bash
sudo ufw default deny incoming
sudo ufw allow from <AUTHORIZED_BMKG_SUBNET> to any port 443 proto tcp
sudo ufw deny 3306/tcp
sudo ufw enable
```

## 4. Office-Specific Items Pending BMKG IT Technician Input
- Official Ubuntu Server IP address & DNS hostname
- Authorized office LAN client subnet(s)
- TLS certificate & private key paths
- Approved server restart & backup retention policy
