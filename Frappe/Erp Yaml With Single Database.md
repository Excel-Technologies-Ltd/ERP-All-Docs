# Erp Yaml With Single Database
**YAML**
```yaml
version: "3.8"

x-customizable-image: &customizable_image
  image: sohanur653/hr-erp:1.0.0
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
      - frappe_overlay_network
    deploy:
      replicas: 1

  redis-cache:
    image: redis:6.2-alpine
    volumes:
      - redis-cache-data:/data
    networks:
      - frappe_overlay_network
    deploy:
      replicas: 1

  redis-queue:
    image: redis:6.2-alpine
    volumes:
      - redis-queue-data:/data
    networks:
      - frappe_overlay_network
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
      - frappe_overlay_network
    deploy:
      replicas: 1
      restart_policy:
        condition: none


  backend:
    <<: *backend_defaults
    networks:
      - frappe_overlay_network
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
      - frappe_overlay_network
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
      - frappe_overlay_network
    deploy:
      replicas: 1

  queue-short:
    <<: *backend_defaults
    command: bench worker --queue short,default
    networks:
      - frappe_overlay_network
    deploy:
      replicas: 1

  queue-long:
    <<: *backend_defaults
    command: bench worker --queue long,default,short
    networks:
      - frappe_overlay_network
    deploy:
      replicas: 1

  scheduler:
    <<: *backend_defaults
    command: bench schedule
    networks:
      - frappe_overlay_network
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
  frappe_overlay_network:
    driver: overlay
    attachable: true
  traefik-public:
    external: true

```

```plain text
DB_PASSWORD=cBmBgnDCkIMZ
SITES=`prod-staff.arcapps.org`
DB_PASSWORD=cBmBgnDCkIMZvat
SITES=`prod-vat.arcapps.org`
excelazmin/erpnext-v14:vat-erp
```
**Site creation**
```shell

bench new-site --mariadb-root-password cBmBgnDCkIMprod --admin-password cBmBgnDCkIMprod --no-mariadb-socket  prod-erp.arcapps.org


bench --site drill-erp.arcapps.org --force restore 20231102_163131-drill-erp_arcapps_org-database.sql.gz --with-private-files 20231102_163131-drill-erp_arcapps_org-private-files.tar --with-public-files 20231102_163131-drill-erp_arcapps_org-files.tar 

bench --site drill-erp.arcapps.org migrate --skip-failing
```
