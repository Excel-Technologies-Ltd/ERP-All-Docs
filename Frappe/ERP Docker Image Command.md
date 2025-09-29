# ERP Docker Image Command
```shell
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
"url": "https://gitlab.com/arcapps/excel_erpnext.git",
"branch": "version-14"
}
]'
```

```shell
export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)
```
```shell
docker buildx build \
  --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg FRAPPE_BRANCH=v14.64.0 \
  --build-arg PYTHON_VERSION=3.11.9 \
  --build-arg NODE_VERSION=18.20.2 \
  --build-arg APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag ghcr.io/user/repo/custom:1.0.0 \
  --file images/custom/Containerfile .
```

**Add Client Script For Asset Doctype**
```javascript
frappe.ui.form.on('Asset', {
	refresh(frm) {
		// your code here
	}
,
	set_values_from_purchase_doc: function (frm, doctype, purchase_doc) {
		frm.set_value("company", purchase_doc.company);
		if (purchase_doc.bill_date) {
			frm.set_value("purchase_date", purchase_doc.bill_date);
		} else {
			frm.set_value("purchase_date", purchase_doc.posting_date);
		}
		if (!frm.doc.is_existing_asset && !frm.doc.available_for_use_date) {
			frm.set_value("available_for_use_date", frm.doc.purchase_date);
		}
		const item = purchase_doc.items.find((item) => item.item_code === frm.doc.item_code);
		if (!item) {
			doctype_field = frappe.scrub(doctype);
			frm.set_value(doctype_field, "");
			frappe.msgprint({
				title: __("Invalid {0}", [__(doctype)]),
				message: __("The selected {0} does not contain the selected Asset Item.", [__(doctype)]),
				indicator: "red",
			});
		}
		frappe.db.get_value("Item", item.item_code, "is_grouped_asset", (r) => {
			var asset_quantity =  1;
			var purchase_amount = flt(
				item.valuation_rate * asset_quantity,
				precision("gross_purchase_amount")
			);

			frm.set_value("gross_purchase_amount", purchase_amount);
			frm.set_value("purchase_receipt_amount", purchase_amount);
			frm.set_value("asset_quantity", asset_quantity);
			frm.set_value("cost_center", item.cost_center || purchase_doc.cost_center);
			if (item.asset_location) {
				frm.set_value("location", item.asset_location);
			}
		});
	},
})
```
