# frappe-bench-v12 - erpsystem.bazrabd.org
**Name**
```plain text
frappe-bench-v12
```

**YAML**
```yaml
version: "3.7"

services:
  redis-cache:
    image: redis:latest
    volumes:
      - redis-cache-vol:/data
    deploy:
      restart_policy:
        condition: on-failure
    networks:
      - frappe-network

  redis-queue:
    image: redis:latest
    volumes:
      - redis-queue-vol:/data
    deploy:
      restart_policy:
        condition: on-failure
    networks:
      - frappe-network

  redis-socketio:
    image: redis:latest
    volumes:
      - redis-socketio-vol:/data
    deploy:
      restart_policy:
        condition: on-failure
    networks:
      - frappe-network

  erpnext-nginx:
    image: registry.gitlab.com/castlecraft/excel_erpnext/excel-erpnext-nginx:${EXCEL_ERPNEXT_VERSION?Variable EXCEL_ERPNEXT_VERSION not set}
    environment:
      - FRAPPE_PY=erpnext-python
      - FRAPPE_PY_PORT=8000
      - FRAPPE_SOCKETIO=frappe-socketio
      - SOCKETIO_PORT=9000
    volumes:
      - sites-vol:/var/www/html/sites:rw
      - assets-vol:/assets:rw
    networks:
      - frappe-network
      - traefik-public
    deploy:
      restart_policy:
        condition: on-failure
      labels:
        - "traefik.docker.network=traefik-public"
        - "traefik.enable=true"
        - "traefik.constraint-label=traefik-public"
        - "traefik.http.routers.erpnext-nginx.rule=Host(${SITES?Variable SITES not set})"
        - "traefik.http.routers.erpnext-nginx.entrypoints=http"
        - "traefik.http.routers.erpnext-nginx.middlewares=https-redirect"
        - "traefik.http.routers.erpnext-nginx-https.rule=Host(${SITES?Variable SITES not set})"
        - "traefik.http.routers.erpnext-nginx-https.entrypoints=https"
        - "traefik.http.routers.erpnext-nginx-https.tls=true"
        - "traefik.http.routers.erpnext-nginx-https.tls.certresolver=le"
        - "traefik.http.services.erpnext-nginx.loadbalancer.server.port=80"

  erpnext-python:
    image: registry.gitlab.com/castlecraft/excel_erpnext/excel-erpnext-worker:${EXCEL_ERPNEXT_VERSION?Variable EXCEL_ERPNEXT_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure
    environment:
      - MARIADB_HOST=${MARIADB_HOST?Variable MARIADB_HOST not set}
      - REDIS_CACHE=redis-cache:6379
      - REDIS_QUEUE=redis-queue:6379
      - REDIS_SOCKETIO=redis-socketio:6379
      - SOCKETIO_PORT=9000
      - AUTO_MIGRATE=1
      - PASS_MAX_DAYS=99999
      - WORKERS=4
    volumes:
      - sites-vol:/home/frappe/frappe-bench/sites:rw
      - assets-vol:/home/frappe/frappe-bench/sites/assets:rw
    networks:
      - frappe-network

  frappe-socketio:
    image: frappe/frappe-socketio:${FRAPPE_VERSION?Variable FRAPPE_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure
    volumes:
      - sites-vol:/home/frappe/frappe-bench/sites:rw
    networks:
      - frappe-network

  erpnext-worker-default:
    image: registry.gitlab.com/castlecraft/excel_erpnext/excel-erpnext-worker:${EXCEL_ERPNEXT_VERSION?Variable EXCEL_ERPNEXT_VERSION not set}
    environment:
      - PASS_MAX_DAYS=9999
    deploy:
      restart_policy:
        condition: on-failure
    command: worker
    volumes:
      - sites-vol:/home/frappe/frappe-bench/sites:rw
    networks:
      - frappe-network

  erpnext-worker-short:
    image: registry.gitlab.com/castlecraft/excel_erpnext/excel-erpnext-worker:${EXCEL_ERPNEXT_VERSION?Variable EXCEL_ERPNEXT_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure
    command: worker
    environment:
      - WORKER_TYPE=short
      - PASS_MAX_DAYS=9999
    volumes:
      - sites-vol:/home/frappe/frappe-bench/sites:rw
    networks:
      - frappe-network

  erpnext-worker-long:
    image: registry.gitlab.com/castlecraft/excel_erpnext/excel-erpnext-worker:${EXCEL_ERPNEXT_VERSION?Variable EXCEL_ERPNEXT_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure
    command: worker
    environment:
      - WORKER_TYPE=long
      - PASS_MAX_DAYS=9999
    volumes:
      - sites-vol:/home/frappe/frappe-bench/sites:rw
    networks:
      - frappe-network

  frappe-schedule:
    image: registry.gitlab.com/castlecraft/excel_erpnext/excel-erpnext-worker:${EXCEL_ERPNEXT_VERSION?Variable EXCEL_ERPNEXT_VERSION not set}
    environment:
      - PASS_MAX_DAYS=9999
    deploy:
      restart_policy:
        condition: on-failure
    command: schedule
    volumes:
      - sites-vol:/home/frappe/frappe-bench/sites:rw
    networks:
      - frappe-network

volumes:
  redis-cache-vol:
  redis-queue-vol:
  redis-socketio-vol:
  assets-vol:
  sites-vol:

networks:
  traefik-public:
    external: true
  frappe-network:
    external: true
```

**Env**
```plain text
SITES=`erpsystem.bazrabd.org`
EXCEL_ERPNEXT_VERSION=v12.19.0
FRAPPE_VERSION=v12.22.2
MARIADB_HOST=frappe-mariadb_mariadb-master
```

```bash
bench new-site --mariadb-root-password o8f3Bg5xwhhMZovBPtk96Ir4s --admin-password o8f3Bg5xwhhMZovBPtk96Ir4s --no-mariadb-socket --force --install-app erpnext erpsystem.bazrabd.org
```


