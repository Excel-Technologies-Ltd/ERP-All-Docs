# Epic 
```yaml
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
   "url": "https://gitlab.com/arcapps/excel_meeting_booking.git",
   "branch": "third-party-integration"
   }
]'

export APPS_JSON_BASE64=$(echo "$APPS_JSON" | base64 -w 0)

docker buildx build \
--no-cache \
--platform linux/amd64,linux/arm64 \
--build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
--build-arg FRAPPE_BRANCH=v14.64.0 \
--build-arg PYTHON_VERSION=3.11.9 \
--build-arg NODE_VERSION=18.20.2 \
--build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
--tag arcapps/epic-erp:1.0.2 \
--file images/custom/Containerfile \
--push .

```
