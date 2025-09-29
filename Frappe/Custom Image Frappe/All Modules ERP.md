# All Modules ERP
```plain text
export APPS_JSON='[
   {
   "url": "https://github.com/frappe/erpnext",
   "branch": "version-14"
   },
   {
   "url": "https://github.com/frappe/hrms.git",
   "branch": "version-14"
   },
   {
   "url": "https://gitlab.com/excel.azmin/whitelabel.git",
   "branch": "main"
   },
   {
   "url": "https://gitlab.com/arcapps/excel_erpnext.git",
   "branch": "version-14"
   }
]
'

export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)

docker buildx build \
--no-cache \
 --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
 --build-arg FRAPPE_BRANCH=version-14 \
 --build-arg PYTHON_VERSION=3.11.9 \
 --build-arg NODE_VERSION=18.20.2 \
 --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
 --tag registry.gitlab.com/arcapps/arc-container-registry/erpnext-v14:v2.0.1 \
 --file images/custom/Containerfile \
 .
&& docker push registry.gitlab.com/arcapps/arc-container-registry/erpnext-v14:v2.0.1
```

```plain text
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
      "url": "https://gitlab.com/arcapps/frappe_survey.git",
      "branch": "master"
   },
   {
      "url": "https://github.com/frappe/education.git",
      "branch": "develop"
   },
   {
      "url": "https://github.com/frappe/lms.git",
      "branch": "v2.15.0"
   },
   {
      "url": "https://github.com/frappe/hospitality.git",
      "branch": "develop"
   },
   {
      "url": "https://gitlab.com/arcapps/excel_meeting_booking.git",
      "branch": "master"
   },
   {
      "url": "https://gitlab.com/arcapps/excel_restaurant_pos.git",
      "branch": "master"
   },
   {
      "url": "https://github.com/frappe/helpdesk.git",
      "branch": "version-14"
   },
   {
      "url": "https://gitlab.com/arcapps/excel-helpdesk.git",
      "branch": "master"
   },
   {
      "url": "https://gitlab.com/arcapps/excel_qr_code.git",
      "branch": "master"
   },
   {
      "url": "https://gitlab.com/arcapps/excel_erpnext.git",
      "branch": "version-14"
   },
    {
      "url": "https://github.com/frappe/payments.git",
      "branch": "version-14"
   },
    {
      "url": "https://github.com/frappe/non_profit.git",
      "branch": "develop"
   }
   
]'


export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)

docker buildx build \
  --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg FRAPPE_BRANCH=version-14 \
  --build-arg PYTHON_VERSION=3.11.9 \
  --build-arg NODE_VERSION=18.20.2 \
  --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
  --tag arcapps/all-erp:dev \
  --file images/custom/Containerfile \
  --push \
  .

```

```bash
bench new-site --mariadb-root-password cBmBgnDCkIMprod --admin-password cBmBgnDCkIMprod --no-mariadb-socket --force  --install-app lms  lms.arcapps.org
```
