# backup-v12 

**Name**
```plain text
backup-v12
```
**YAML**
```YAML
version: "3.7"

services:
  backup:
    image: registry.gitlab.com/castlecraft/excel_erpnext/excel-erpnext-worker:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    command: backup
    environment:
      - WITH_FILES=1
    volumes:
      - "sites-vol:/home/frappe/frappe-bench/sites"
    deploy:
      labels:
        - "swarm.cronjob.enable=true"
        - "swarm.cronjob.schedule=0 0 * * *"
        - "swarm.cronjob.skip-running=true"
      replicas: 0
      restart_policy:
        condition: none
    networks:
      - frappe-network

  push-backup:
    image: registry.gitlab.com/castlecraft/excel_erpnext/excel-erpnext-worker:${ERPNEXT_VERSION?Variable ERPNEXT_VERSION not set}
    command: push-backup
    environment:
      - BUCKET_NAME=excelerp-backups
      - REGION=ap-southeast-1
      - ACCESS_KEY_ID=AKIAZYOAF7WBUPXRMFN6
      - SECRET_ACCESS_KEY=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
      - ENDPOINT_URL=https://s3.ap-southeast-1.amazonaws.com
      - BUCKET_DIR=staffportainer-bench-v12
      - BACKUP_LIMIT=10
    volumes:
      - "sites-vol:/home/frappe/frappe-bench/sites"
    deploy:
      labels:
        - "swarm.cronjob.enable=true"
        - "swarm.cronjob.schedule=30 0 * * *"
        - "swarm.cronjob.skip-running=true"
      replicas: 0
      restart_policy:
        condition: none
    networks:
      - frappe-network

volumes:
  sites-vol:
    external: true
    name: frappe-bench-v12_sites-vol

networks:
  frappe-network:
    external: true
```

**Env**
```plain text
ERPNEXT_VERSION=v12.0.12
```
