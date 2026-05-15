# Amor Fati Personal AI Infrastructure

This repository documents the migration of `amorfati.cn` from a traditional WordPress deployment to a Docker-first, reverse-proxy-friendly, AI-infra-ready server architecture.

The goal is not only to host a personal website, but to gradually evolve the server into a lightweight self-hosted AI systems platform for:

- personal website / portfolio
- research notes
- WordPress content
- FastAPI backend
- future Agent services
- MCP services
- Open WebUI
- n8n / workflow automation
- LangGraph / AI orchestration
- future memory / vector database services

---

## 1. Final Architecture

Current production architecture:

```text
Internet
    |
DNS
    |
Caddy HTTPS Reverse Proxy
    |
Docker network: infra
    |-- WordPress Nginx
    |       |
    |   WordPress FPM
    |       |
    |   MySQL 5.7
    |
    `-- FastAPI Service
```

Current public routes:

```text
https://amorfati.cn              -> WordPress
https://amorfati.cn/wp-login.php -> WordPress admin login
https://amorfati.cn/api/health   -> FastAPI health check
https://amorfati.cn/api/docs     -> FastAPI Swagger docs
```

---

## 2. Server Environment

Current server:

```text
Provider: Tencent Cloud Lighthouse
Region: Overseas
OS: OpenCloudOS 9.4
Architecture: Docker-first
Reverse proxy: Caddy
Website runtime: WordPress FPM + Nginx + MySQL
API runtime: FastAPI
```

Important design decisions:

```text
No Baota / aaPanel
No Apache
No bare-metal WordPress
No host-level Certbot
No host-level Nginx as the public entrypoint
```

All production services are intended to be managed by Docker Compose.

---

## 3. Current Directory Layout

Main application root:

```text
/opt/apps
|-- caddy
|-- wordpress
|-- api
`-- _shared
    |-- env
    |-- logs
    `-- volumes
```

Recommended future layout:

```text
/opt/apps
|-- caddy
|-- wordpress
|-- api
|-- openwebui
|-- mcp
|-- n8n
|-- langgraph
|-- redis
|-- qdrant
`-- _shared
    |-- env
    |-- logs
    |-- volumes
    `-- backups
```

---

## 4. Docker Network

A shared Docker network is used for internal service communication:

```bash
docker network create infra
```

All future AI services should join the same `infra` network unless isolation is required.

Current principle:

```text
Expose only Caddy to the host network.
Keep backend services internal.
```

---

## 5. Current Containers

Current active containers:

```text
amorfati_caddy       -> Caddy HTTPS reverse proxy
amorfati_nginx       -> Nginx for WordPress static/PHP routing
amorfati_wordpress   -> WordPress FPM
amorfati_mysql       -> MySQL 5.7
amorfati_api         -> FastAPI backend
```

Expected checks:

```bash
docker ps
docker network ls
docker network inspect infra
```

---

## 6. Public Entry Point

Caddy owns the public ports:

```text
80/tcp
443/tcp
```

The host should not run another Nginx, Apache, Baota, or Certbot service that competes for ports 80/443.

Expected listener model:

```text
80/443 -> docker-proxy -> amorfati_caddy
```

Check:

```bash
ss -tunlp | grep -E ':80|:443'
```

---

## 7. HTTPS

HTTPS is handled by Caddy.

Caddy automatically:

- obtains Let's Encrypt certificates
- renews certificates
- redirects HTTP to HTTPS
- reverse proxies traffic to backend containers

Expected validation:

```bash
curl -I https://amorfati.cn
curl -I https://amorfati.cn/wp-login.php
curl -s https://amorfati.cn/api/health
```

Expected results:

```text
https://amorfati.cn              -> HTTP 200
https://amorfati.cn/wp-login.php -> HTTP 200
https://amorfati.cn/api/health   -> healthy
```

---

## 8. WordPress Migration Summary

The old WordPress site was migrated from the previous server.

Old site information:

```text
Old WordPress root: /www/wwwroot/<old-domain>
Old database: <old-wordpress-database>
Table prefix: wp_
```

Migration principle:

Only migrate:

```text
1. MySQL database
2. wp-content
```

Do not migrate:

```text
Baota config
Old Nginx config
Old SSL certificates
Old PHP runtime
Old Apache/Nginx host environment
WordPress core files if the new Docker WordPress image already provides them
```

---

## 9. Migration Backups

Before migration, the new server was backed up.

Created backups included:

```text
/opt/apps_before_wp_migration_<timestamp>.tar.gz
/opt/<current-wordpress-db-backup>.sql.gz
/opt/<wordpress-content-volume-backup>.tar.gz
/opt/<wordpress-mysql-volume-backup>.tar.gz
```

Old server export files:

```text
/root/<old-wordpress-db-export>.sql.gz
/root/<old-wordpress-content-export>.tar.gz
```

New server migration packages:

```text
/opt/apps/<old-wordpress-db-export>.sql.gz
/opt/apps/<old-wordpress-content-export>.tar.gz
```

Old new-site `wp-content` backup:

```text
/var/lib/docker/volumes/wordpress_wordpress_data/_data/wp-content.before_migration_20260514_151555
```

Do not delete these immediately.

Recommended retention:

```text
Keep old server and migration backups for at least 3-7 days.
```

---

## 10. WordPress URL Fix

After migration, WordPress URLs were updated to:

```text
home    = https://amorfati.cn
siteurl = https://amorfati.cn
```

Old URL remnants were checked in:

```text
wp_options
wp_posts
wp_postmeta
```

Checked and cleaned patterns:

```text
<old-server-ip>
<old-domain>
http://amorfati.cn:8080
```

Residual count was confirmed as:

```text
0
```

---

## 11. Post-Migration Validation

Validation commands:

```bash
curl -I https://amorfati.cn
curl -I https://amorfati.cn/wp-login.php
curl -s https://amorfati.cn/api/health
docker ps
```

Expected:

```text
WordPress homepage: HTTP 200
WordPress login:    HTTP 200
FastAPI health:     healthy
MySQL:              healthy
```

Current container status:

```text
amorfati_api         healthy
amorfati_caddy       up
amorfati_nginx       up
amorfati_wordpress   up
amorfati_mysql       healthy
```

---

## 12. Manual Browser Checks

After migration, manually verify:

```text
Homepage rendering
Images and media library
Theme layout
Plugins
wp-admin login
Permalinks
Article pages
Mobile layout
Upload function
```

Recommended WordPress action:

```text
wp-admin -> Settings -> Permalinks -> Save Changes
```

This refreshes WordPress rewrite rules.

---

## 13. Why Baota Was Removed

Baota was intentionally removed from the new server.

Reason:

The new server uses a Docker-first infrastructure. Keeping Baota would introduce a second, competing control plane.

Risks avoided:

```text
Baota Nginx competing with Caddy
Baota SSL competing with Caddy
Port 80/443 conflicts
Cron/systemd leftovers
Unexpected rewrite/config overwrite
Harder Docker Compose maintenance
Agent infra incompatibility
```

Before removal, it was verified that:

```text
80/443 were handled by Docker/Caddy
WordPress was inside Docker
MySQL was inside Docker
FastAPI was inside Docker
Docker volumes were independent
```

Special handling:

Baota had placed swap under `/www/swap`.

Before deleting `/www`, swap was safely migrated to:

```text
/swapfile
```

and `/etc/fstab` was updated.

---

## 14. Firewall Policy

Current minimal firewall policy:

```text
22/tcp   SSH
80/tcp   HTTP
443/tcp  HTTPS
```

Removed legacy ports:

```text
8888/tcp  Baota panel
8080/tcp  temporary WordPress test port
```

Check:

```bash
firewall-cmd --list-ports
ss -tunlp
```

---

## 15. Performance Notes

Current server memory is limited:

```text
2C2G
```

Current stack is acceptable for:

```text
WordPress
FastAPI
Caddy
small API services
lightweight agent experiments
```

Potential bottlenecks:

```text
WordPress plugins
MySQL memory
PHP-FPM workers
VSCode Remote extensionHost
Docker container overhead
swap usage
```

Useful checks:

```bash
free -h
htop
docker stats
curl -o /dev/null -s -w "%{time_total}\n" https://amorfati.cn
```

If WordPress feels slow, first check:

```text
1. Heavy WordPress plugins
2. MySQL memory pressure
3. PHP-FPM saturation
4. VSCode Remote memory usage
5. Swap activity
6. Overseas network latency
```

Do not immediately blame Caddy or Docker.

---

## 16. Recommended Next Steps

Priority order:

```text
1. Stabilize current migrated WordPress site for 24-72 hours
2. Confirm browser rendering and wp-admin behavior
3. Create a post-migration production snapshot
4. Add structured logs
5. Add Redis
6. Turn FastAPI into an AI gateway layer
7. Add Qdrant for vector memory
8. Add MCP services
9. Add Open WebUI
10. Add n8n / LangGraph / Agent workers
```

Do not deploy everything at once.

---

## 17. Future AI Gateway Plan

FastAPI should gradually evolve from a demo API into an AI gateway layer.

Planned endpoints:

```text
/api/health
/api/status
/api/metrics
/api/chat
/api/memory
/api/embeddings
/api/agent
/api/workflow
/api/eeg
```

Future service layers:

```text
FastAPI Gateway
|
Agent Runtime
|
MCP Tools
|
Memory / Vector DB
|
External APIs / Research Tools / EEG Services
```

---

## 18. Future AI Infra Components

Recommended future services:

| Service | Role |
|---|---|
| Redis | cache, queue, session, lightweight memory |
| Qdrant | vector database / semantic memory |
| Open WebUI | LLM interaction layer |
| MCP server | tool ecosystem |
| n8n | workflow automation |
| LangGraph | agent orchestration |
| FastAPI | gateway / backend API |
| WordPress | public content and research portal |

---

## 19. Operating Principles

Keep the architecture:

```text
Docker-first
Reverse-proxy-friendly
AI-infra-friendly
Future-agent-extensible
Lightweight
Maintainable
```

Avoid:

```text
Baota
Apache
Host-level Certbot
Host-level Nginx as public entry
Manual production edits without backup
Hardcoded secrets in compose files
Exposing internal services directly
```

---

## 20. Useful Commands

Check containers:

```bash
docker ps
```

Check logs:

```bash
docker logs amorfati_caddy --tail=100
docker logs amorfati_nginx --tail=100
docker logs amorfati_wordpress --tail=100
docker logs amorfati_mysql --tail=100
docker logs amorfati_api --tail=100
```

Check public site:

```bash
curl -I https://amorfati.cn
curl -I https://amorfati.cn/wp-login.php
curl -s https://amorfati.cn/api/health
```

Check listening ports:

```bash
ss -tunlp
```

Check memory:

```bash
free -h
docker stats
```

Backup `/opt/apps`:

```bash
cd /opt
tar -czvf apps_snapshot_$(date +%Y%m%d_%H%M%S).tar.gz apps
```

Backup WordPress database:

```bash
docker exec amorfati_mysql mysqldump -u root -p wordpress | gzip > /opt/final_migrated_wp_$(date +%Y%m%d_%H%M%S).sql.gz
```

Note: replace `wordpress` with the actual database name if different.

---

## 21. Current Status

Current status:

```text
Migration completed.
Docker-first infra active.
Caddy HTTPS active.
WordPress migrated.
FastAPI active.
Baota removed.
Apache not used.
Bare-metal WordPress not used.
Ready for AI service expansion.
```

Current public site:

```text
https://amorfati.cn
```
