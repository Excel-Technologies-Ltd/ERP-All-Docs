# Care Pro LMS Image
```plain text
export APPS_JSON='[

{
 "name": "care_pro_lms",
"url": "https://gitlab.com/arcapps/care_pro_lms.git",
"branch": "carepro"
},
{
"url": "https://gitlab.com/arcapps/excel_lms.git",
"branch": "carepro"
}
]'

export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)

docker build \
		--no-cache \
    --platform linux/amd64 \
    --build-arg FRAPPE_PATH=https://github.com/Excel-Technologies-Ltd/Frappe-LMS.git \
    --build-arg FRAPPE_BRANCH=version-15 \
    --build-arg PYTHON_VERSION=3.11.9 \
    --build-arg NODE_VERSION=18.20.2 \
    --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
    --tag arcapps/lms:1.0.0 \
    --file images/custom/Containerfile \
    --push \
   .

```
