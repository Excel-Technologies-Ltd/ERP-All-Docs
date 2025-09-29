# frappe-mariadb
**Name**
```plain text
frappe-mariadb
```

**YAML**
```yaml
version: "3.7"

services:
  mariadb-master:
    image: 'bitnami/mariadb:10.3'
    deploy:
      restart_policy:
        condition: on-failure
    networks:
      - frappe-network
    volumes:
      - 'mariadb_master_data:/bitnami/mariadb'
    environment:
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
      - MARIADB_EXTRA_FLAGS=--skip-character-set-client-handshake
      - MARIADB_REPLICATION_MODE=master
      - MARIADB_REPLICATION_USER=repl_user
      - MARIADB_REPLICATION_PASSWORD=o8f3Bg5xwhhMZovBPtk96Ir4s
      - MARIADB_ROOT_PASSWORD=o8f3Bg5xwhhMZovBPtk96Ir4s

  mariadb-slave:
    image: 'bitnami/mariadb:10.3'
    deploy:
      restart_policy:
        condition: on-failure
    networks:
      - frappe-network
    volumes:
      - 'mariadb_slave_data:/bitnami/mariadb'
    environment:
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
      - MARIADB_EXTRA_FLAGS=--skip-character-set-client-handshake
      - MARIADB_REPLICATION_MODE=slave
      - MARIADB_REPLICATION_USER=repl_user
      - MARIADB_REPLICATION_PASSWORD=o8f3Bg5xwhhMZovBPtk96Ir4s
      - MARIADB_MASTER_HOST=mariadb-master
      - MARIADB_MASTER_PORT_NUMBER=3306
      - MARIADB_MASTER_ROOT_PASSWORD=o8f3Bg5xwhhMZovBPtk96Ir4s

volumes:
  mariadb_master_data:
  mariadb_slave_data:

networks:
  frappe-network:
    name: frappe-network
    attachable: true
```

**Env**
