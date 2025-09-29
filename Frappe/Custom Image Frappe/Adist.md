# Adist
```powershell
export APPS_JSON='[
   {
   "url": "https://github.com/frappe/erpnext",
   "branch": "v14.60.1"
   },
   {
   "url": "https://github.com/frappe/hrms.git",
   "branch": "v14.21.0"
   },
   {
   "url": "https://gitlab.com/excel.azmin/whitelabel.git",
   "branch": "main"
   },
   {
   "url": "https://github.com/Excel-Technologies-Ltd/Adist.git",
   "branch": "main"
   },
   {
   "url": "https://github.com/Excel-Technologies-Ltd/Fastrack.git",
   "branch": "main"
   }
   
]
'

export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)

docker buildx build \
--no-cache \
--platform linux/amd64 \
 --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
 --build-arg FRAPPE_BRANCH=v14.64.0 \
 --build-arg PYTHON_VERSION=3.11.9 \
 --build-arg NODE_VERSION=18.20.2 \
 --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
 --tag arcapps/ftcl_adist:dev \
 --file images/custom/Containerfile \
 --push .
```
