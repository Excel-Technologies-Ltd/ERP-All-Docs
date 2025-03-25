# Mongodb-replica

# Create a new stack `**mongodb_replica**`

```yaml
version: "3.8"
services:
  mongodb-master:
    image: mongo:6.0.8
    deploy:
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "2"
          memory: "6G"
    volumes:
      - mongodb-master:/data/db
      - dump:/data/dump
    networks:
      - mongodb_replica_network
    depends_on:
      - mongodb-slave1
      - mongodb-slave2
    links:
      - mongodb-slave1
      - mongodb-slave2
    ports:
      - "27017:27017"
    environment:
      - MONGODB_USERNAME=rma-server
      - MONGODB_PASSWORD=mOngoRootPass2024
      - MONGODB_DATABASE=rma-server
      - MONGODB_ROOT_PASSWORD=mOngoRootPass2024
      - MONGODB_PRIMARY_ROOT_USER=root
      - CACHE_DB=cache-db
      - CACHE_USER=cache-db
      - CACHE_DB_PASSWORD=mOngoRootPass2024
      - SERVER_DB=rma-server
      - SERVER_USER=rma-server
      - SERVER_DB_PASSWORD=mOngoRootPass2024
    entrypoint:
      [
        "/usr/bin/mongod",
        "--bind_ip_all",
        "--replSet",
        "mongodb_replica_set",
        "--wiredTigerCacheSizeGB",
        "6",
      ]

  mongodb-slave1:
    image: mongo:6.0.8
    deploy:
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "2"
          memory: "6G"
    volumes:
      - mongodb-slave1:/data/db
    ports:
      - "27016:27017"
    networks:
      - mongodb_replica_network
    entrypoint:
      [
        "/usr/bin/mongod",
        "--bind_ip_all",
        "--replSet",
        "mongodb_replica_set",
        "--wiredTigerCacheSizeGB",
        "6",
      ]

  mongodb-slave2:
    image: mongo:6.0.8
    deploy:
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "2"
          memory: "6G"
    volumes:
      - mongodb-slave2:/data/db

    networks:
      - mongodb_replica_network
    entrypoint:
      [
        "/usr/bin/mongod",
        "--bind_ip_all",
        "--replSet",
        "mongodb_replica_set",
        "--wiredTigerCacheSizeGB",
        "6",
      ]

networks:
  mongodb_replica_network:
    name: mongodb_replica_network
    attachable: true

volumes:
  mongodb-master:
  mongodb-slave1:
  mongodb-slave2:
  dump:
```

# Replication setup

go to mongodb_replica stack and open container shell in `mongodb-master`

make sure container name is mongodb-master

```bash
mongosh
```

```bash
rs.initiate({_id: 'mongodb_replica_set',members: [{_id: 0, host: 'mongodb-master:27017'},{_id: 1, host: 'mongodb-slave1:27017'}, {_id: 2, host: 'mongodb-slave2:27017'}]});
conf = rs.config()
conf.members[0].priority = 2;
rs.reconfig(conf)
rs.status()
```

# DB Restoreation

```bash
cd /opt
mkdir restore
cd ./restore

apt update
apt install curl

curl https://files-for-excel-bd.s3.ap-southeast-1.amazonaws.com/Oct+28+Main+ERP/20231028000000.tar.gz --output dump.tar.gz
tar xvfz dump.tar.gz
rm -fr /dump/admin
mongorestore --db rma-server dump/rma-server/

```

# Create DB User

```bash
mongosh
use cache-db
db.createUser({user: 'cache-db', pwd: 'mOngoRootPass2024', roles:[{role:'dbOwner', db: 'cache-db'}], passwordDigestor: 'server'});
use rma-server
db.createUser({user: 'rma-server', pwd: "mOngoRootPass2024", roles:[{role:'dbOwner', db: 'rma-server'}], passwordDigestor: 'server'});
```
