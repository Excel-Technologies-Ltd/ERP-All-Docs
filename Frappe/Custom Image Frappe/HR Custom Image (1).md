# HR Custom Image (1)

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
"url": "https://gitlab.com/arcapps/excel_hr.git",
"branch": "master"
},
{
"url": "https://gitlab.com/arcapps/excel_meeting_booking.git",
"branch": "master"
}
,
{
"url": "https://gitlab.com/arcapps/excel_attendance.git",
"branch": "ivms"
}
,
{
"url": "https://gitlab.com/arcapps/excel_qr_code.git",
"branch": "master"
}
,
{
"url": "https://gitlab.com/arcapps/excel-helpdesk.git",
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
 --tag sohanur653/hr-erp:1.0.0 \
 --file images/custom/Containerfile \
 . && docker push sohanur653/hr-erp:1.0.0
```
