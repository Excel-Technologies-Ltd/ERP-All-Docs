# Setup ErpNext

### Create a Stack With `Erpnext` name

### Enter this Yaml

```yaml
version: "3.8"

x-customizable-image: &customizable_image
  image: registry.gitlab.com/arcapps/arc-container-registry/erpnext-v14:${ERP_VERSION?Variable ERP_VERSION not set}
  # image: arcapps/erp:1.0.2
  deploy:
    update_config:
      parallelism: 1
    rollback_config:
      parallelism: 1

x-backend-defaults: &backend_defaults
  <<: [*customizable_image]
  volumes:
    - sites:/home/frappe/frappe-bench/sites
  deploy:
    replicas: 1
    update_config:
      parallelism: 1
    rollback_config:
      parallelism: 1

services:
  db-master:
    image: mariadb:10.6
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD:?No db password set}
      MYSQL_DATABASE: ${DB_NAME:-frappe}
      MYSQL_USER: ${DB_USER:-frappe}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --skip-character-set-client-handshake
      - --skip-innodb-read-only-compressed # Temporary fix for MariaDB 10.6
    volumes:
      - db-data-master:/var/lib/mysql
    networks:
      - frappe_erp_overlay_network
      - frappe_network
    ports:
      - "3306:3306"
    deploy:
      replicas: 1

  redis-cache:
    image: redis:6.2-alpine
    volumes:
      - redis-cache-data:/data
    networks:
      - frappe_erp_overlay_network
    deploy:
      replicas: 1

  redis-queue:
    image: redis:6.2-alpine
    volumes:
      - redis-queue-data:/data
    networks:
      - frappe_erp_overlay_network
    deploy:
      replicas: 1

  configurator:
    <<: *backend_defaults
    entrypoint:
      - bash
      - -c
    command:
      - >
        ls -1 apps > sites/apps.txt;
        bench set-config -g db_host $$DB_HOST;
        bench set-config -gp db_port $$DB_PORT;
        bench set-config -g redis_cache "redis://$$REDIS_CACHE";
        bench set-config -g redis_queue "redis://$$REDIS_QUEUE";
        bench set-config -g redis_socketio "redis://$$REDIS_QUEUE";
        bench set-config -gp socketio_port $$SOCKETIO_PORT;
    environment:
      DB_HOST: db-master
      DB_PORT: 3306
      REDIS_CACHE: redis-cache:6379
      REDIS_QUEUE: redis-queue:6379
      SOCKETIO_PORT: 9000
    networks:
      - frappe_erp_overlay_network
    deploy:
      replicas: 1
      restart_policy:
        condition: none

  backend:
    <<: *backend_defaults
    networks:
      - frappe_erp_overlay_network
    deploy:
      replicas: 1

  frontend:
    <<: *customizable_image
    command:
      - nginx-entrypoint.sh
    environment:
      BACKEND: backend:8000
      FRAPPE_SITE_NAME_HEADER: $$host
      SOCKETIO: websocket:9000
      UPSTREAM_REAL_IP_ADDRESS: ${UPSTREAM_REAL_IP_ADDRESS:-127.0.0.1}
      UPSTREAM_REAL_IP_HEADER: ${UPSTREAM_REAL_IP_HEADER:-X-Forwarded-For}
      UPSTREAM_REAL_IP_RECURSIVE: ${UPSTREAM_REAL_IP_RECURSIVE:-off}
      PROXY_READ_TIMEOUT: ${PROXY_READ_TIMEOUT:-120}
      CLIENT_MAX_BODY_SIZE: ${CLIENT_MAX_BODY_SIZE:-50m}
    volumes:
      - sites:/home/frappe/frappe-bench/sites
    networks:
      - frappe_erp_overlay_network
      - traefik-public
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
        delay: 5s
      labels:
        traefik.docker.network: traefik-public
        traefik.enable: "true"
        traefik.constraint-label: traefik-public
        traefik.http.routers.staff--http.rule: Host(${SITES:?No sites set})
        traefik.http.routers.staff--http.entrypoints: http
        traefik.http.routers.staff--http.middlewares: https-redirect
        traefik.http.routers.staff--https.rule: Host(${SITES})
        traefik.http.routers.staff--https.entrypoints: https
        traefik.http.routers.staff--https.tls: "true"
        traefik.http.routers.staff--https.tls.certresolver: le
        traefik.http.services.staff-.loadbalancer.server.port: "8080"
    depends_on:
      - backend
      - websocket

  websocket:
    <<: [*customizable_image]
    command:
      - node
      - /home/frappe/frappe-bench/apps/frappe/socketio.js
    volumes:
      - sites:/home/frappe/frappe-bench/sites
    networks:
      - frappe_erp_overlay_network
    deploy:
      replicas: 1

  queue-short:
    <<: *backend_defaults
    command: bench worker --queue short,default
    networks:
      - frappe_erp_overlay_network
    deploy:
      replicas: 1

  queue-long:
    <<: *backend_defaults
    command: bench worker --queue long,default,short
    networks:
      - frappe_erp_overlay_network
    deploy:
      replicas: 1

  scheduler:
    <<: *backend_defaults
    command: bench schedule
    networks:
      - frappe_erp_overlay_network
    deploy:
      replicas: 1

volumes:
  sites:
  db-data-master:
  db-data-slave1:
  db-data-slave2:
  redis-cache-data:
  redis-queue-data:

networks:
  frappe_erp_overlay_network:
    driver: overlay
    attachable: true
  traefik-public:
    external: true
  frappe_network:
    external: false
```

# ENV

Select `Advance Mode` paste this

```bash
DB_PASSWORD=J9PYsUrhMIzay8Farcone
SITES=`arcone-erp.arcapps.org`
ERP_VERSION=v2.0.4
```

# After successfully create a stack now make a site

```bash
bench new-site --mariadb-root-password VTtVNLTvqi0K9tl --admin-password VTtVNLTvqi0K9tl --no-mariadb-socket --force --install-app erpnext --install-app hrms --install-app excel_erpnext  drill-erp.arcapps.org
```

# Restore database

### Download mariadb backup from asw s3

```bash
wget https://files-for-excel-bd.s3.ap-southeast-1.amazonaws.com/March+8+Main+ERP+2025/frappe-bench-v14-backup-2025-03-08_00-30-00.tar.gz

```

Now extract this (open shell as a root user)

```bash
tar xvfz frappe-bench-v14-backup-2025-03-08_00-30-00.tar.gz
```

after extract this, database path will

```bash
data/bd.bazrabd.org/private/backup/20231102_163131-drill-erp_arcapps_org-database.sql.gz
```

### with database

```bash
bench --site drill-erp.arcapps.org --force restore data/bd.bazrabd.org/private/backup/20231102_163131-drill-erp_arcapps_org-database.sql.gz
```

### with database and fils

```bash
bench --site drill-erp.arcapps.org --force restore 20231102_163131-drill-erp_arcapps_org-database.sql.gz --with-private-files 20231102_163131-drill-erp_arcapps_org-private-files.tar --with-public-files 20231102_163131-drill-erp_arcapps_org-files.tar
```

### Migrate this site with database

```bash
bench --site drill-erp.arcapps.org migrate --skip-failing
```
