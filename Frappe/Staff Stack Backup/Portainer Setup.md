# Setup command for portainer
**Starting the docker swarm installation**
```bash
export USE_HOSTNAME=arcone-docker.arcapps.org
echo $USE_HOSTNAME > /etc/hostname
hostname -F /etc/hostname
apt update && apt upgrade -y
apt install curl
curl -fsSL get.docker.com -o get-docker.sh
CHANNEL=stable sh get-docker.sh
rm get-docker.sh
docker swarm init
docker ps
sudo groupadd docker
sudo usermod -aG docker $USER

docker swarm init --advertise-addr=202.91.42.199
```
**Traefik Proxy with HTTPS**
```bash
docker network create --driver=overlay traefik-public
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
docker node update --label-add traefik-public.traefik-public-certificates=true $NODE_ID
export EMAIL=mark@arcapps.com
export DOMAIN=mark-traefik.arcapps.org
export USERNAME=admin
export PASSWORD=J9PYsUrhMIzay8Fmark
export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
curl -L dockerswarm.rocks/traefik-host.yml -o traefik-host.yml
docker stack deploy -c traefik-host.yml traefik
docker stack ps traefik
```
**Portainer web user interface**
```bash
export DOMAIN=mark-portainer.arcapps.org
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
docker node update --label-add portainer.portainer-data=true $NODE_ID
curl -L dockerswarm.rocks/portainer.yml -o portainer.yml
docker stack deploy -c portainer.yml portainer
docker stack ps portainer
```
```plain text
J9PYsUrhMIzay8Farcone
```
```javascript
db.createUser({user: 'cache-db', pwd: 'mOngoRootPass2024', roles:[{role:'dbOwner', db: 'cache-db'}], passwordDigestor: 'server'});
db.createUser({user: 'rma-server', pwd: "mOngoRootPass2024", roles:[{role:'dbOwner', db: 'rma-server'}], passwordDigestor: 'server'});
```
