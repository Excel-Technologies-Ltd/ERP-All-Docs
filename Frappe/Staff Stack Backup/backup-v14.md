# backup-v14 
**Name**
```plain text
backup-v14
```
**YAML**
```yaml

version: "3.7"

services:
  backup:
    image: excelazmin/hr-erp:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    entrypoint: ["bash", "-c"]
    command:
      - |
        bench --site staff.bazrabd.org backup --with-files
    volumes:
      - "sites:/home/frappe/frappe-bench/sites"
    deploy:
      labels:
        - "swarm.cronjob.enable=true"
        - "swarm.cronjob.schedule=0 0 * * *"
        - "swarm.cronjob.skip-running=true"
      replicas: 0
      restart_policy:
        condition: none
    networks:
      - frappe-bench-db-network
  backup-push:
    image: excelazmin/docker-s3-cron-backup:v1.0.0
    environment:
      - AWS_ACCESS_KEY_ID=AKIAZYOAF7WB5YICBMXV
      - AWS_SECRET_ACCESS_KEY=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
      - S3_BUCKET_URL=s3://main-erp-backups/staff.bazrabd.org/
      - AWS_DEFAULT_REGION=ap-southeast-1
      - CRON_SCHEDULE=30 0 * * *
      - BACKUP_NAME=frappe-bench-v14-backup
    volumes:
      - "sites:/data:ro"
    deploy:
      labels:
        - "swarm.cronjob.enable=true"
        - "swarm.cronjob.schedule=0 0 * * *"
        - "swarm.cronjob.skip-running=true"
      replicas: 0

      restart_policy:
        condition: none
    networks:
      - frappe-bench-db-network
networks:
  frappe-bench-db-network:
    name: v14erp-db-network
    external: true
volumes:
  sites:
    external: true
    name: erpnext-v14_sites
```

**Env**
```plain text
ERPNEXT_VERSION=v14.73.0
```
