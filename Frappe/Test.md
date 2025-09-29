# Test

**YAML**
```yaml
version: "3.7"

services:
  rma-frontend:
    image: registry.gitlab.com/arcapps/excel-rma/rma-frontend:${RMA_FRONTEND_VERSION?Variable RMA_FRONTEND_VERSION not set}
    environment:
      - API_HOST=rma-server
      - API_PORT=8800
    networks:
      - excel_mongo_dev
      - traefik-public
    deploy:
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - node.labels.frappe-node.data == true 
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=traefik-public"
        - "traefik.constraint-label=traefik-public"
        - "traefik.http.routers.craft-rma-frontend.rule=Host(`${SITE?Variable SITE not set}`)"
        - "traefik.http.routers.craft-rma-frontend.entrypoints=http"
        - "traefik.http.routers.craft-rma-frontend.middlewares=https-redirect"
        - "traefik.http.middlewares.craft-rma-frontend.headers.contentTypeNosniff=true"
        - "traefik.http.routers.craft-rma-frontend-https.rule=Host(`${SITE?Variable SITE not set}`)"
        - "traefik.http.routers.craft-rma-frontend-https.entrypoints=https"
        - "traefik.http.routers.craft-rma-frontend-https.tls=true"
        - "traefik.http.routers.craft-rma-frontend-https.tls.certresolver=le"
        - "traefik.http.services.craft-rma-frontend.loadbalancer.server.port=8080"

  rma-warranty:
    image: registry.gitlab.com/arcapps/excel-rma/rma-warranty:${RMA_WARRANTY_VERSION?Variable RMA_WARRANTY_VERSION not set}
    environment:
      - API_HOST=rma-server
      - API_PORT=8800
    networks:
      - excel_mongo_dev
      - traefik-public
    deploy:
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - node.labels.frappe-node.data == true  
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=traefik-public"
        - "traefik.constraint-label=traefik-public"
        - "traefik.http.routers.craft-rma-warranty.rule=Host(`${WARRANTY_SITE?Variable WARRANTY_SITE not set}`)"
        - "traefik.http.routers.craft-rma-warranty.entrypoints=http"
        - "traefik.http.routers.craft-rma-warranty.middlewares=https-redirect"
        - "traefik.http.middlewares.craft-rma-warranty.headers.contentTypeNosniff=true"
        - "traefik.http.routers.craft-rma-warranty-https.rule=Host(`${WARRANTY_SITE?Variable WARRANTY_SITE not set}`)"
        - "traefik.http.routers.craft-rma-warranty-https.entrypoints=https"
        - "traefik.http.routers.craft-rma-warranty-https.tls=true"
        - "traefik.http.routers.craft-rma-warranty-https.tls.certresolver=le"
        - "traefik.http.services.craft-rma-warranty.loadbalancer.server.port=8080"

  rma-server:
    image: registry.gitlab.com/arcapps/excel-rma/rma-server:${RMA_SERVER_VERSION?Variable RMA_SERVER_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - node.labels.frappe-node.data == true 
    healthcheck:	
      test: (echo >/dev/tcp/rma-server/8800) &>/dev/null && exit 0 || exit 1	
      interval: 1s	
      retries: 15
    environment:
      - DB_HOST=mongo1
      - DB_NAME=rma-server
      - DB_PASSWORD=u4MDujth69EkVNJs0i
      - DB_USER=rma-server
      - CACHE_DB_NAME=cache-db
      - CACHE_DB_PASSWORD=u4MDujth69EkVNJs0i
      - CACHE_DB_USER=cache-db
      - NODE_ENV=production
      - NODE_OPTIONS='--max-old-space-size=4096 --prof'
      - AGENDA_JOBS_CONCURRENCY=1
      - REPLICA_SET_NAME=excel_rs_dev
      - MONGO_REPLICA_PRIMARY=mongo1
      - MONGO_REPLICA_SECONDARY_1=mongo2
      - MONGO_REPLICA_SECONDARY_2=mongo3
      - MONGO_REPLICA_PRIMARY_PORT=27017
      - MONGO_REPLICA_SECONDARY_1_PORT=27017
      - MONGO_REPLICA_SECONDARY_2_PORT=27017
      - READ_PREFERENCE=secondaryPreferred
    networks:
      -  excel_mongo_dev

networks:
  traefik-public:
    external: true
  excel_mongo_dev:
    external: true
```

```plain text
RMA_FRONTEND_VERSION=v2.0.3
RMA_WARRANTY_VERSION=v2.0.3
RMA_SERVER_VERSION=v2.0.3
SITE=dev-etlrma.arcapps.org
WARRANTY_SITE=dev-etlwarranty.arcapps.org
```
