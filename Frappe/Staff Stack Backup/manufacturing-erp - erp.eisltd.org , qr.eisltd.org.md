# manufacturing-erp - erp.eisltd.org , qr.eisltd.org

**Name**
```plain text
manufacturing-erp
```

**YAML**
```yaml

version: "3.7"

services:
  backend:
    image: excelazmin/manufacturing-erp:${ERP_VERSION?Variable ERP_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure  
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
    networks:  
      - manufacturing-erp-db-network
      - manufacturing-erp-bench-network  
  
  configurator:
    image: excelazmin/manufacturing-erp:${ERP_VERSION?Variable ERP_VERSION not set}
    deploy:
      restart_policy:
        condition: none  
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
        bench set-config -g redis_socketio "redis://$$REDIS_SOCKETIO";
        bench set-config -gp socketio_port $$SOCKETIO_PORT;
    environment:
      DB_HOST: mariadb-master
      DB_PORT: "3306"
      REDIS_CACHE: redis-cache:6379
      REDIS_QUEUE: redis-queue:6379
      REDIS_SOCKETIO: redis-socketio:6379
      SOCKETIO_PORT: "9000"
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs

  mariadb-master:
    image: docker.io/bitnami/mariadb:11.1
    deploy:
      restart_policy:
        condition: on-failure
    volumes:
      - 'mariadb_master_data:/bitnami/mariadb'
    environment:
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
      - MARIADB_EXTRA_FLAGS=--skip-character-set-client-handshake
      - MARIADB_PASSWORD=oHY8kNc2qKuuNJTKPjpk
      - MARIADB_REPLICATION_MODE=master
      - MARIADB_REPLICATION_USER=repl_user
      - MARIADB_USER=my_user
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=No
      - MARIADB_ROOT_PASSWORD=oHY8kNc2qKuuNJTKPjpk
      - MARIADB_REPLICATION_PASSWORD=oHY8kNc2qKuuNJTKPjpk
    networks:  
      - manufacturing-erp-db-network 
    healthcheck:
      test: ['CMD', '/opt/bitnami/scripts/mariadb/healthcheck.sh']
      interval: 15s
      timeout: 5s
      retries: 6

  mariadb-slave:
    image: docker.io/bitnami/mariadb:11.1
    deploy:
      restart_policy:
        condition: on-failure
    depends_on:
      - mariadb-master
    volumes:
      - 'mariadb_slave_data:/bitnami/mariadb'
    environment:
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
      - MARIADB_EXTRA_FLAGS=--skip-character-set-client-handshake    
      - MARIADB_REPLICATION_MODE=slave
      - MARIADB_REPLICATION_USER=repl_user
      - MARIADB_PASSWORD=oHY8kNc2qKuuNJTKPjpk
      - MARIADB_USER=my_user
      - ALLOW_EMPTY_PASSWORD=No
      - MARIADB_MASTER_HOST=mariadb-master
      - MARIADB_MASTER_PORT_NUMBER=3306
      - MARIADB_MASTER_ROOT_PASSWORD=oHY8kNc2qKuuNJTKPjpk
      - MARIADB_REPLICATION_PASSWORD=oHY8kNc2qKuuNJTKPjpk
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
    networks:  
      - manufacturing-erp-db-network 
    healthcheck:
      test: ['CMD', '/opt/bitnami/scripts/mariadb/healthcheck.sh']
      interval: 15s
      timeout: 5s
      retries: 6
  
  frontend:
    image: excelazmin/manufacturing-erp:${ERP_VERSION?Variable ERP_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure  
      labels:
        traefik.docker.network: traefik-public
        traefik.enable: "true"
        traefik.constraint-label: traefik-public
        # Change router name prefix from erpnext- to the name of stack in case of multi bench setup
        traefik.http.routers.erpnext-testing--http.rule: Host(${SITES:?No sites set})
        traefik.http.routers.erpnext-testing--http.entrypoints: http
        # Remove following lines in case of local setup
        traefik.http.routers.erpnext-testing--http.middlewares: https-redirect
        traefik.http.routers.erpnext-testing--https.rule: Host(${SITES})
        traefik.http.routers.erpnext-testing--https.entrypoints: https
        traefik.http.routers.erpnext-testing--https.tls: "true"
        traefik.http.routers.erpnext-testing--https.tls.certresolver: le
        # Remove above lines in case of local setup
        traefik.http.services.erpnext-testing-.loadbalancer.server.port: "8080"    

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
      - manufacturing-erp-db-network
      - manufacturing-erp-bench-network
      - traefik-public 

  queue-default:
    image: excelazmin/manufacturing-erp:${ERP_VERSION?Variable ERP_VERSION not set}
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
      - manufacturing-erp-db-network
      - manufacturing-erp-bench-network   

  queue-long:
    image: excelazmin/manufacturing-erp:${ERP_VERSION?Variable ERP_VERSION not set}
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
      - manufacturing-erp-db-network
      - manufacturing-erp-bench-network   

  queue-short:
    image: excelazmin/manufacturing-erp:${ERP_VERSION?Variable ERP_VERSION not set}
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
      - manufacturing-erp-db-network
      - manufacturing-erp-bench-network   

  redis-queue:
    image: redis:6.2-alpine
    deploy:
      restart_policy:
        condition: on-failure  
    volumes:
      - redis-queue-data:/data
    networks:  
      - manufacturing-erp-bench-network  

  redis-cache:
    image: redis:6.2-alpine
    deploy:
      restart_policy:
        condition: on-failure  
    volumes:
      - redis-cache-data:/data
    networks:  
      - manufacturing-erp-bench-network    

  redis-socketio:
    image: redis:6.2-alpine
    deploy:
      restart_policy:
        condition: on-failure  
    volumes:
      - redis-socketio-data:/data
    networks:  
      - manufacturing-erp-bench-network  

  scheduler:
    image: excelazmin/manufacturing-erp:${ERP_VERSION?Variable ERP_VERSION not set}
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
      - manufacturing-erp-db-network
      - manufacturing-erp-bench-network   

  websocket:
    image: excelazmin/manufacturing-erp:${ERP_VERSION?Variable ERP_VERSION not set}
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
      - manufacturing-erp-db-network
      - manufacturing-erp-bench-network   
      
volumes:
  db-data:
  redis-queue-data:
  redis-cache-data:
  redis-socketio-data:
  sites:
  logs:
  mariadb_master_data:
  mariadb_slave_data:

networks:
  manufacturing-erp-bench-network:
    name: manufacturing-erp-bench-network
    external: false
  manufacturing-erp-db-network:
    name: manufacturing-erp-db-network
    external: true
  traefik-public:
    name: traefik-public
    external: true  

```
**ENV**
```plain text
ERP_VERSION=v14.70.6
SITES=`erp.eisltd.org`,`qr.eisltd.org`
```

```bash
bench new-site --mariadb-root-password oHY8kNc2qKuuNJTKPjpk --admin-password oHY8kNc2qKuuNJTKPjpk --no-mariadb-socket --force --install-app erpnext qr.eisltd.org
```
