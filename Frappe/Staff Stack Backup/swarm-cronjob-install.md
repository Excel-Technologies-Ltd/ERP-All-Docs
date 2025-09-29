# swarm-cronjob-install
**Name**
```plain text
swarm-cronjob-install
```
**YAML**
```yaml
version: "3.2"

services:
  swarm-cronjob:
    image: crazymax/swarm-cronjob
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    environment:
      - "TZ=Aisa/Dhaka"
      - "LOG_LEVEL=info"
      - "LOG_JSON=false"
```
