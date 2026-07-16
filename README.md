# Nextcloud + OnlyOffice Docker Deployment

Production Ready Deployment menggunakan Docker Compose.

---

# Environment

| Component | Version |
|-----------|---------|
| Ubuntu Server | 24.04 LTS |
| Docker Engine | Latest Stable |
| Docker Compose | v2 |
| Nextcloud | 34.0.1 Apache |
| MariaDB | 11.x |
| Redis | 7.x |
| OnlyOffice Document Server | Latest Stable |

---

# Architecture

```

Internet/Internal
│
├──────────────┐
│              │
Nextcloud OnlyOffice
│              │
├──────┐
│      │
MariaDB Redis

```

---

# Docker Volumes

```

/srv/nextcloud
├── app
├── config
├── custom_apps
├── data
├── db
├── redis
└── compose
    ├── docker-compose.yml
    └── .env

```

Semua data bersifat persistent.

Upgrade container **tidak akan menghapus data**.

---

# Nextcloud Configuration

Redis digunakan untuk:

- File Locking
- Distributed Cache
- Local Cache

config.php

```
docker exec -it nextcloud bash
php occ config:system:set memcache.local --value='\OC\Memcache\APCu'
php occ config:system:set memcache.locking --value='\OC\Memcache\Redis'
php occ config:system:set redis host --value=redis
php occ maintenance:repair

```

---

# Trusted Domains

```
192.168.12.29:8081
nextcloud
onlyoffice
172.18.0.5
```

---

# OnlyOffice Integration

mkdir -p \
/srv/nextcloud/onlyoffice/data \
/srv/nextcloud/onlyoffice/logs \
/srv/nextcloud/onlyoffice/lib

Integrated using:

```
ONLYOFFICE Connector
```

Status
✅ Working
Document editing works.

---

# Database

MariaDB
Separate container.
No SQLite used.

---

# Background Jobs

Mode

```
Cron
```

Recommended.

---

# Docker Restart Policy

```
restart: unless-stopped
```

---

# Backup Strategy

Backup required

- MariaDB Dump
- Nextcloud Data
- Config
- Docker Compose
- OnlyOffice Data

Recommended schedule

Daily

---

# Upgrade Procedure

Backup database

```
mysqldump ...
```

Pull latest image

```
docker compose pull
```

Recreate container

```
docker compose up -d
```

Run OCC Upgrade

```
docker exec -u www-data nextcloud php occ upgrade
```

---

# Installed Apps

Core

- Files
- Calendar
- Contacts
- Notes
- Mail
- Activity
- Dashboard

Office

- OnlyOffice
- Richdocuments

Administration

- Admin Audit
- Server Info

---

# External Storage

Current Status
SMB Server
✅ Working
Command Line Test

```
smbclient //192.168.12.29/"Support Service" -U it
```

Result
Success
Nextcloud GUI
❌ Failed
Error

```
Network settings prevented the connection
```

---

# Investigation

Verified
✅ SMB Server
✅ Authentication
✅ Share Access
✅ User Permission
✅ Samba Configuration
✅ www-data access
✅ smbclient binary
Verified Command

```
su -s /bin/bash www-data

smbclient //192.168.12.29/"Support Service" -U it
```

Success.

---

# Current Issue

Nextcloud

```
files_external:verify
```

returns

```
status : error
code : 1
message :
```

without any exception.
No NT_STATUS error returned.

---

# Root Cause Analysis

Possibility
Nextcloud Docker Image

```
nextcloud:34.0.1-apache
```

Running on

- Debian 13
- PHP 8.4

Image does not contain

```
php-smbclient
```

Only binary

```
/usr/bin/smbclient
```

is available.
Current hypothesis
Possible bug on SMB fallback backend in Nextcloud 34.

---

# Pending Improvement

- Compile php-smbclient
- Create custom Docker image
- Re-test SMB External Storage

---

# Production Checklist

| Item | Status |
|------|--------|
| Docker Compose | ✅ |
| Persistent Volume | ✅ |
| Redis | ✅ |
| MariaDB | ✅ |
| Nextcloud | ✅ |
| OnlyOffice | ✅ |
| Backup Strategy | ✅ |
| Upgrade Tested | ✅ |
| Reverse Proxy Ready | ✅ |
| External Storage SMB | ⚠ Pending |

---

# Future Improvements

- HTTPS
- Automatic Backup
- BorgBackup
- Restic
- ClamAV
- Preview Generator
- Prometheus Exporter
- Grafana Dashboard
- Loki Log Monitoring
- SSO
- LDAP Integration

---

# Useful OCC Commands

Maintenance

```
Aktifkan maintenance mode:
docker exec -u www-data nextcloud php occ maintenance:mode --on

Recreate container:
docker compose up -d --force-recreate nextcloud

Matikan maintenance mode:
docker exec -u www-data nextcloud php occ maintenance:mode --off
```

```bash
# Compress directory folder
sudo tar -czvf nextcloud-prd.tar.gz /srv/nextcloud 

sudo rsync -avhP --info=progress2 \
-e "ssh -p 22" \
/srv/nextcloud/nextcloud-prd.tar.gz \
user@IP:/home/user/

# Verification permission
sudo chown -R 33:33 /srv/nextcloud/data
sudo chown -R 999:999 /srv/nextcloud/db

# Kalau IP berubah
cukup edit
config.php

# atau
docker exec -u www-data nextcloud php occ status
docker exec -u www-data nextcloud php occ config:system:set trusted_domains 4 --value=192.168.105.9
docker exec nextcloud php occ config:list system
```

Upgrade

```
docker exec -u www-data nextcloud php occ upgrade
```

Repair

```
docker exec -u www-data nextcloud php occ maintenance:repair
```

Apps

```
docker exec -u www-data nextcloud php occ app:list
```

External Storage

```
docker exec -u www-data nextcloud php occ files_external:list

docker exec -u www-data nextcloud php occ files_external:verify <id>
```

Logs

```
docker exec -u www-data nextcloud php occ log:watch
```

---

---

Maintainer

```
Verdi Verdian
System Administrator
```

Last Updated

```
July 2026
```
