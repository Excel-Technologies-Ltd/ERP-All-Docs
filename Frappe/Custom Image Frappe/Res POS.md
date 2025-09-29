# Res POS
```plain text
export APPS_JSON='[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-14"
  },
  {
    "url": "https://gitlab.com/arcapps/excel_restaurant_pos.git",
    "branch": "master"
  },
  {
    "url": "https://github.com/NagariaHussain/doppio.git",
    "branch": "master"
  },
  {
"url": "https://gitlab.com/arcapps/excel_qr_code.git",
"branch": "master"
}
]'

export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)

docker build \
    --no-cache \
    --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
    --build-arg FRAPPE_BRANCH=version-14 \
    --build-arg PYTHON_VERSION=3.11.9 \
    --build-arg NODE_VERSION=18.20.2 \
    --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
    --tag arcapps/res-pos:latest \
    --file images/custom/Containerfile \
    . && docker push arcapps/res-pos:latest

```
