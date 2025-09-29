# CRM Image
```plain text
export APPS_JSON='[
{
"url": "https://github.com/frappe/erpnext",
"branch": "version-15"
},
{
"url": "https://github.com/frappe/hrms.git",
"branch": "version-15"
},
{
"url": "https://github.com/frappe/crm.git",
"branch": "v1.26.0"
},
{
"url": "https://github.com/frappe/helpdesk.git",
"branch": "v1.1.0"
},
{
"url": "https://gitlab.com/arcapps/frappe_survey.git",
"branch": "master"
}
]'

export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)

docker build \
	--no-cache \
 --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
 --build-arg FRAPPE_BRANCH=version-15 \
 --build-arg PYTHON_VERSION=3.11.9 \
 --build-arg NODE_VERSION=18.20.2 \
 --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
 --tag sohanur653/hr-crm:1.0.0 \
 --file images/custom/Containerfile \
 . && docker push sohanur653/hr-crm:1.0.0
```
