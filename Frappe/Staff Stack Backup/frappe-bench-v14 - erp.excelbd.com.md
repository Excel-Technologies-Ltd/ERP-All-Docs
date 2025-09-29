# frappe-bench-v14 - erp.excelbd.com

**Name**
```plain text
frappe-bench-v14
```
**YAML**
```yaml

version: "3.7"

services:
  backend:
    image: excelazmin/erpnext-v14:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure  
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:  
      - frappe-bench-db-network
      - frappe-bench-network  

  mariadb-master:
    image: docker.io/bitnami/mariadb:10.6
    deploy:  
      restart_policy:
        condition: on-failure
    volumes:
      - 'mariadb_master_data:/bitnami/mariadb'
    environment:
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
      - MARIADB_EXTRA_FLAGS=--skip-character-set-client-handshake
      - MARIADB_PASSWORD=uI07LV8AhvU6BT6u6o3g
      - MARIADB_REPLICATION_MODE=master
      - MARIADB_REPLICATION_USER=repl_user
      - MARIADB_USER=my_user
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=No
      - MARIADB_ROOT_PASSWORD=uI07LV8AhvU6BT6u6o3g
      - MARIADB_REPLICATION_PASSWORD=uI07LV8AhvU6BT6u6o3g
    networks:  
      - frappe-bench-db-network 
    healthcheck:
      test: ['CMD', '/opt/bitnami/scripts/mariadb/healthcheck.sh']
      interval: 15s
      timeout: 5s
      retries: 6

  mariadb-slave1:
    image: docker.io/bitnami/mariadb:10.6
    deploy:  
      restart_policy:
        condition: on-failure
    depends_on:
      - mariadb-master
    volumes:
      - 'mariadb_slave1_data:/bitnami/mariadb'
    environment:
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
      - MARIADB_EXTRA_FLAGS=--skip-character-set-client-handshake    
      - MARIADB_REPLICATION_MODE=slave
      - MARIADB_REPLICATION_USER=repl_user
      - MARIADB_PASSWORD=uI07LV8AhvU6BT6u6o3g
      - MARIADB_USER=my_user
      - ALLOW_EMPTY_PASSWORD=No
      - MARIADB_MASTER_HOST=mariadb-master
      - MARIADB_MASTER_PORT_NUMBER=3306
      - MARIADB_MASTER_ROOT_PASSWORD=uI07LV8AhvU6BT6u6o3g
      - MARIADB_REPLICATION_PASSWORD=uI07LV8AhvU6BT6u6o3g
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
    networks:  
      - frappe-bench-db-network 
    healthcheck:
      test: ['CMD', '/opt/bitnami/scripts/mariadb/healthcheck.sh']
      interval: 15s
      timeout: 5s
      retries: 6
  frontend:
    image: excelazmin/erpnext-v14:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure  
      labels:
        traefik.docker.network: traefik-public
        traefik.enable: "true"
        traefik.constraint-label: traefik-public
        # Change router name prefix from erpnext-v13 to the name of stack in case of multi bench setup
        traefik.http.routers.frappe-bench-v14-http.rule: Host(${SITES:?No sites set})
        traefik.http.routers.frappe-bench-v14-http.entrypoints: http
        # Remove following lines in case of local setup
        traefik.http.routers.frappe-bench-v14-http.middlewares: https-redirect
        traefik.http.routers.frappe-bench-v14-https.rule: Host(${SITES})
        traefik.http.routers.frappe-bench-v14-https.entrypoints: https
        traefik.http.routers.frappe-bench-v14-https.tls: "true"
        traefik.http.routers.frappe-bench-v14-https.tls.certresolver: le
        # Remove above lines in case of local setup
        traefik.http.services.frappe-bench-v14.loadbalancer.server.port: "8080"    

    command:
      - nginx-entrypoint.sh
    environment:
      BACKEND: backend:8000
      FRAPPE_SITE_NAME_HEADER: $$host
      SOCKETIO: websocket:9000
      UPSTREAM_REAL_IP_ADDRESS: 127.0.0.1
      UPSTREAM_REAL_IP_HEADER: X-Forwarded-For
      UPSTREAM_REAL_IP_RECURSIVE: "off"
      PROXY_READ_TIMEOUT: 120
      CLIENT_MAX_BODY_SIZE: 50m
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:
      - frappe-bench-db-network
      - frappe-bench-network
      - traefik-public 

  queue-default:
    image: excelazmin/erpnext-v14:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure  
    command:
      - bench
      - worker
      - --queue
      - default
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:  
      - frappe-bench-db-network
      - frappe-bench-network   

  queue-long:
    image: excelazmin/erpnext-v14:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    deploy:  
      restart_policy:
        condition: on-failure  
    command:
      - bench
      - worker
      - --queue
      - long
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:  
      - frappe-bench-db-network
      - frappe-bench-network   

  queue-short:
    image: excelazmin/erpnext-v14:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    deploy:  
      restart_policy:
        condition: on-failure  
    command:
      - bench
      - worker
      - --queue
      - short
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:  
      - frappe-bench-db-network
      - frappe-bench-network   

  redis-queue:
    image: redis:6.2-alpine
    deploy:
      restart_policy:
        condition: on-failure  
    volumes:
      - redis-queue-data:/data
    networks:  
      - frappe-bench-network  

  redis-cache:
    image: redis:6.2-alpine
    deploy:  
      restart_policy:
        condition: on-failure  
    volumes:
      - redis-cache-data:/data
    networks:  
      - frappe-bench-network    

  redis-socketio:
    image: redis:6.2-alpine
    deploy:  
      restart_policy:
        condition: on-failure  
    volumes:
      - redis-socketio-data:/data
    networks:  
      - frappe-bench-network  

  scheduler:
    image: excelazmin/erpnext-v14:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    deploy:  
      restart_policy:
        condition: on-failure  
    command:
      - bench
      - schedule
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:  
      - frappe-bench-db-network
      - frappe-bench-network   

  websocket:
    image: excelazmin/erpnext-v14:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    deploy:  
      restart_policy:
        condition: on-failure  
    command:
      - node
      - /home/frappe/frappe-bench/apps/frappe/socketio.js
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:  
      - frappe-bench-db-network
      - frappe-bench-network     
      
volumes:
  db-data:
  redis-queue-data:
  redis-cache-data:
  redis-socketio-data:
  sites:
  logs:
  mariadb_master_data:
  mariadb_slave1_data:


networks:
  frappe-bench-network:
    name: frappe-bench-network
    external: false
  frappe-bench-db-network:
    name: frappe-bench-db-network
    external: true
  traefik-public:
    name: traefik-public
    external: true

```

**Env**
```plain text
SITES=`erp.excelbd.com`
ERPNEXT_VERSION=vat-erp
```

```bash
bench new-site --mariadb-root-password uI07LV8AhvU6BT6u6o3g --admin-password uI07LV8AhvU6BT6u6o3g --no-mariadb-socket --force --install-app erpnext erp.excelbd.com
```
