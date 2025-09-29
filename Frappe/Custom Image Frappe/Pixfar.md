# Pixfar
```json
export APPS_JSON='[
{
"url": "https://github.com/frappe/erpnext",
"branch": "v14.70.7"
}
]'

export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)

docker buildx build \
 --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
 --build-arg FRAPPE_BRANCH=v14.64.0 \
 --build-arg PYTHON_VERSION=3.11.9 \
 --build-arg NODE_VERSION=18.20.2 \
 --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
 --tag sohanur653/erpnext:1.0.0 \
 --file images/custom/Containerfile \
 .
docker push sohanur653/erpnext:1.0.0
```
