# Setup Rma-Server

# Create stack with name `rma-server`

```yaml
version: "3.7"

services:
  rma-frontend:
    image: registry.gitlab.com/arcapps/excel-rma/rma-frontend:${RMA_FRONTEND_VERSION?Variable RMA_FRONTEND_VERSION not set}
    environment:
      - API_HOST=rma-server
      - API_PORT=8800
    networks:
      - mongodb_replica_network
      - traefik-public
    deploy:
      restart_policy:
        condition: on-failure
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=traefik-public"
        - "traefik.constraint-label=traefik-public"
        - "traefik.http.routers.arc-rma-frontend.rule=Host(`${SITE?Variable SITE not set}`)"
        - "traefik.http.routers.arc-rma-frontend.entrypoints=http"
        - "traefik.http.routers.arc-rma-frontend.middlewares=https-redirect"
        - "traefik.http.middlewares.arc-rma-frontend.headers.contentTypeNosniff=true"
        - "traefik.http.routers.arc-rma-frontend-https.rule=Host(`${SITE?Variable SITE not set}`)"
        - "traefik.http.routers.arc-rma-frontend-https.entrypoints=https"
        - "traefik.http.routers.arc-rma-frontend-https.tls=true"
        - "traefik.http.routers.arc-rma-frontend-https.tls.certresolver=le"
        - "traefik.http.services.arc-rma-frontend.loadbalancer.server.port=8080"

  rma-warranty:
    image: registry.gitlab.com/arcapps/excel-rma/rma-warranty:${RMA_WARRANTY_VERSION?Variable RMA_WARRANTY_VERSION not set}
    environment:
      - API_HOST=rma-server
      - API_PORT=8800
    networks:
      - mongodb_replica_network
      - traefik-public
    deploy:
      restart_policy:
        condition: on-failure
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=traefik-public"
        - "traefik.constraint-label=traefik-public"
        - "traefik.http.routers.arc-rma-warranty.rule=Host(`${WARRANTY_SITE?Variable WARRANTY_SITE not set}`)"
        - "traefik.http.routers.arc-rma-warranty.entrypoints=http"
        - "traefik.http.routers.arc-rma-warranty.middlewares=https-redirect"
        - "traefik.http.middlewares.arc-rma-warranty.headers.contentTypeNosniff=true"
        - "traefik.http.routers.arc-rma-warranty-https.rule=Host(`${WARRANTY_SITE?Variable WARRANTY_SITE not set}`)"
        - "traefik.http.routers.arc-rma-warranty-https.entrypoints=https"
        - "traefik.http.routers.arc-rma-warranty-https.tls=true"
        - "traefik.http.routers.arc-rma-warranty-https.tls.certresolver=le"
        - "traefik.http.services.arc-rma-warranty.loadbalancer.server.port=8080"

  rma-server:
    image: registry.gitlab.com/arcapps/excel-rma/rma-server:${RMA_SERVER_VERSION?Variable RMA_WARRANTY_VERSION not set}
    deploy:
      restart_policy:
        condition: on-failure
    healthcheck:
      test: (echo >/dev/tcp/rma-server/8800) &>/dev/null && exit 0 || exit 1
      interval: 1s
      retries: 15
    environment:
      - DB_HOST=mongodb-master
      - DB_NAME=rma-server
      - DB_PASSWORD=mOngoRootPass2024
      - DB_USER=rma-server
      - CACHE_DB_NAME=cache-db
      - CACHE_DB_PASSWORD=mOngoRootPass2024
      - CACHE_DB_USER=cache-db
      - NODE_ENV=production
      - NODE_OPTIONS='--max-old-space-size=4096 --prof'
      - AGENDA_JOBS_CONCURRENCY=1
      - REPLICA_SET_NAME=mongodb_replica_set
      - MONGO_REPLICA_PRIMARY=mongodb-master
      - MONGO_REPLICA_SECONDARY_1=mongodb-slave1
      - MONGO_REPLICA_SECONDARY_2=mongodb-slave2
      - MONGO_REPLICA_PRIMARY_PORT=27017
      - MONGO_REPLICA_SECONDARY_1_PORT=27017
      - MONGO_REPLICA_SECONDARY_2_PORT=27017
      - READ_PREFERENCE=secondaryPreferred
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - MAIL_HOST=smtp.zoho.com
      - MAIL_PORT=465
      - MAIL_SECURE=true
      - MAIL_IGNORE_TLS=true
      - MAIL_USER=notifications@excelbd.com
      - MAIL_PASSWORD=ETL&aliV$P@783
      - MAIL_SENDER_NAME=ArApps
      - SMS_API_KEY=6ebbe5ba6570301d
      - SMS_SENDER_NAME=ArcApps
      - SMS_API_URL=http://smpp.revesms.com:7788/sendtext
      - SMS_SECRET_KEY=f2aa2bdc
      - SMS_CALLER_ID=574943
    networks:
      - mongodb_replica_network
  redis:
    image: redis:latest

    volumes:
      - redis-data:/data # Persist Redis data
    restart: unless-stopped
    networks:
      - mongodb_replica_network
networks:
  traefik-public:
    external: true
  mongodb_replica_network:
    external: true

volumes:
  redis-data:
```

# ENV

```bash
RMA_FRONTEND_VERSION=v1.0.0
RMA_SERVER_VERSION=v1.0.0
RMA_WARRANTY_VERSION=v1.0.0
SITE=arcone-rma.arcapps.org
WARRANTY_SITE=arcone-warranty.arcapps.org
```
