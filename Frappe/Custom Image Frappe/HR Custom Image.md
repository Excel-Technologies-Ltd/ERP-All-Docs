# HR Custom Image
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
"url": "https://github.com/Excel-Technologies-Ltd/Excel-HR.git",
"branch": "master"
},
{
"url": "https://gitlab.com/arcapps/excel_meeting_booking.git",
"branch": "master"
}
,
{
"url": "https://github.com/Excel-Technologies-Ltd/Excel-Attendance.git",
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

docker buildx build \
	--no-cache \
	--build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
	--build-arg FRAPPE_BRANCH=version-14 \
	--build-arg PYTHON_VERSION=3.11.9 \
	--build-arg NODE_VERSION=18.20.2 \
	--build-arg APPS_JSON_BASE64=${APPS_JSON_BASE64} \
	--tag registry.gitlab.com/arcapps/arc-container-registry/hr-erp:1.0.8 \
	--file images/custom/Containerfile \
	--push .

```

```plain text
arcapps/hr-erp:1.0.0
```

```powershell
nohup bash -c "
export APPS_JSON='[
{
  \"url\": \"https://github.com/frappe/erpnext\",
  \"branch\": \"version-14\"
},
{
  \"url\": \"https://github.com/frappe/hrms.git\",
  \"branch\": \"version-14\"
},
{
  \"url\": \"https://gitlab.com/excel.azmin/whitelabel.git\",
  \"branch\": \"main\"
},
{
  \"url\": \"https://gitlab.com/arcapps/excel_hr.git\",
  \"branch\": \"master\"
},
{
  \"url\": \"https://gitlab.com/arcapps/excel_meeting_booking.git\",
  \"branch\": \"master\"
},
{
  \"url\": \"https://gitlab.com/arcapps/excel_attendance.git\",
  \"branch\": \"ivms\"
},
{
  \"url\": \"https://gitlab.com/arcapps/excel_qr_code.git\",
  \"branch\": \"master\"
},
{
  \"url\": \"https://gitlab.com/arcapps/excel-helpdesk.git\",
  \"branch\": \"master\"
}
]'
export APPS_JSON_BASE64=\$(echo \${APPS_JSON} | base64 -w 0)
docker buildx build --no-cache \
	--platform linux/amd64,linux/arm64 \
  --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg FRAPPE_BRANCH=version-14 \
  --build-arg PYTHON_VERSION=3.11.9 \
  --build-arg NODE_VERSION=18.20.2 \
  --build-arg APPS_JSON_BASE64=\${APPS_JSON_BASE64} \
  --tag arcapps/hr-erp:latest \
  --file images/custom/Containerfile \
  --push
  . 
" > build.log 2>&1 &

```
