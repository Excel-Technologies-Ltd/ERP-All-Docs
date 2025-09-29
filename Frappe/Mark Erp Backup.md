# Mark Erp Backup

**Portal Backend Setup**

```bash
https://mark-rma.arcapps.org/api/direct/callback http://rma.localhost:4700/api/direct/callback 

https://mark-rma.arcapps.org/api/direct/callback
```

**Portal Frontend Client**

```bash
io.identityserver.demo:/oauthredirect https://mark-rma.arcapps.org/callback https://mark-rma.arcapps.org/silent-refresh.html https://mark-warranty.arcapps.org/callback https://mark-warranty.arcapps.org/silent-refresh.html http://rma.localhost:4700/callback http://rma.localhost:4700/silent-refresh.html



https://mark-rma.arcapps.org/callback
```

**Server Settings Backup**
```javascript
{
  _id: ObjectId("600c02e7165b02000df96bb1"),
  service: 'rma-server',
  uuid: '939d9d3c-d4f6-4dad-8192-dfe4dd64e7d2',
  appURL: 'https://mark-rma.arcapps.org',
  warrantyAppURL: 'https://mark-warranty.arcapps.org',
  frontendClientId: '1cae8c4f76',
  backendClientId: 'fe6df0a17b',
  serviceAccountUser: 'bot@excelbd.com',
  serviceAccountSecret: 'Admin123635356',
  authServerURL: 'https://mark-erp.arcapps.org',
  serviceAccountApiKey: '403dfa271ac2021',
  serviceAccountApiSecret: 'ae97ee879198e07',
  profileURL: 'https://mark-erp.arcapps.org/api/method/frappe.integrations.oauth2.openid_profile',
  authorizationURL: 'https://mark-erp.arcapps.org/api/method/frappe.integrations.oauth2.authorize',
  tokenURL: 'https://mark-erp.arcapps.org/api/method/frappe.integrations.oauth2.get_token',
  revocationURL: 'https://mark-erp.arcapps.org/api/method/frappe.integrations.oauth2.revoke_token',
  scope: [ 'all', 'openid' ],
  webhookApiKey: 'b9fcc4c8c1be9b6681c6ec20e2c9935ef2f8c450b9728f7c7e00bbfc38076a00f3740cdbad884c97d64db24389e15dae0156590ee840ffa54d3348fd827d79d0',
  timeZone: 'Asia/Dhaka',
  defaultCompany: 'Excel Technologies Ltd.',
  sellingPriceList: 'Standard Selling',
  debtorAccount: '10203 - Accounts Receivable - ETL',
  transferWarehouse: 'Work In Progress - ETL',
  validateStock: true,
  footerImageURL: 'https://bd.bazrabd.org/files/SINV-Footer.jpg',
  footerWidth: 530,
  headerImageURL: 'https://bd.bazrabd.org/files/ETL-header.jpg',
  headerWidth: 500,
  brand: { faviconURL: 'https://bd.bazrabd.org/files/Fav-Icon.png' },
  backdatedInvoices: true,
  posAppURL: 'http://pos.localhost:4900',
  backdatedInvoicesForDays: 165,
  posProfile: 'Service Center',
  brandWiseCreditLimit: true,
  displayCategory: true,
  displayTax: true,
  displayPromotionalOffer: true,
  displayPaymentData: true,
  displayStock: true
}
```

**Asset Client Script**
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
**Payment Entry Backup**
```javascript
function validate_brand_wise_payments(frm) {
  let paidAmount = frm.doc.paid_amount;

  if (!paidAmount) {
    msgprint("Paid amount can't be empty");
    return false;
  }

  if (!frm.doc.custom_brand_wise_payments) {
    return true;
  }

  let brand_wise_payments_sum = 0;

  for (let i = 0; i < frm.doc.custom_brand_wise_payments.length; i++) {
    brand_wise_payments_sum += frm.doc.custom_brand_wise_payments[i].amount;
  }

  if (brand_wise_payments_sum > paidAmount) {
    msgprint('The sum of brand-wise payments cannot exceed the paid amount.');
    return false;
  }

  return true;
}

frappe.ui.form.on('Payment Entry', 'validate', function (frm) {
  frappe.validated = validate_brand_wise_payments(frm);
});

frappe.ui.form.on('Payment Entry', 'before_submit', function (frm) {
  if (frm.doc.mode_of_payment === 'Cheque in Hand') {
    msgprint('Cheque in Hand can not be submitted');
    frappe.validated = false;
    return;
  }
});

frappe.ui.form.on('Payment Entry', 'before_submit', function(frm) {
        if (frm.doc.mode_of_payment === 'Dishonour Cheque') {
            msgprint('Dishonour Cheque can not be submitted');
            frappe.validated = false;
        }
});

function updateBrandWiseAllocations(frm, isCancellation) {
  const updateLimit = (allocation, amount) => {
    allocation.limit = isCancellation
      ? allocation.limit - amount
      : allocation.limit + amount;
  };

  if (frm.doc.party_type === 'Customer') {
    frappe.db.get_doc('Customer', frm.doc.party).then((doc) => {
      const brandWiseAllocations = doc.custom_brand_wise_allocations;
      let brandWiseTotalAmount = 0;

      if (brandWiseAllocations && brandWiseAllocations.length > 0) {
        for (const brandWisePayment of frm.doc.custom_brand_wise_payments) {
          const brand = brandWisePayment.brand;
          const amount = brandWisePayment.amount;

          const brandWiseAllocation = brandWiseAllocations.find(
            (x) => x.brand === brand
          );
          if (brandWiseAllocation) {
            updateLimit(brandWiseAllocation, amount);
            brandWiseTotalAmount += amount;
          }
        }
      }

      console.log('isCancellation', isCancellation);

      const otherBrandsLimit = frm.doc.paid_amount - brandWiseTotalAmount;
      console.log('otherBrandsLimit', otherBrandsLimit);

      const excel_remaining_balance = !isCancellation
        ? doc.excel_remaining_balance + frm.doc.paid_amount
        : doc.excel_remaining_balance - frm.doc.paid_amount;

      console.log(
        'excel_remaining_balance',
        doc.excel_remaining_balance,
        excel_remaining_balance
      );

      const custom_other_brands_limit = !isCancellation
        ? doc.custom_other_brands_limit + otherBrandsLimit
        : doc.custom_other_brands_limit - otherBrandsLimit;

      console.log(
        'custom_other_brands_limit',
        doc.custom_other_brands_limit,
        custom_other_brands_limit
      );

      frappe.db
        .set_value('Customer', frm.doc.party, {
          excel_remaining_balance,
          custom_other_brands_limit,
          custom_brand_wise_allocations: doc.custom_brand_wise_allocations,
          custom_source: frm.doc.name,
        })
        .then((res) => {
          console.log('Update response', res.message);
        });
    });
  }
}

frappe.ui.form.on('Payment Entry', 'after_cancel', function (frm) {
  updateBrandWiseAllocations(frm, true);
});

frappe.ui.form.on('Payment Entry', 'on_submit', function (frm) {
  updateBrandWiseAllocations(frm, false);
});

//Sales Person & customer contact
frappe.ui.form.on('Payment Entry', {
	refresh:function(frm) {
		frm.toggle_reqd("excel_bank_name_cheque",frm.doc.mode_of_payment==="Cheque" || frm.doc.mode_of_payment==="Cheque In Hand" || frm.doc.mode_of_payment==="Dishonour Cheque")
	}
})

frappe.ui.form.on('Payment Entry', {
	party:function(frm) {
	    const isTrue = frm.doc.party_type=="Customer" ? true : false
		if(isTrue && frm.doc.party){
		       frappe.call({
                method: 'frappe.client.get_value',
                args: {
                    doctype: 'Customer',
                    filters: { name: frm.doc.party },
                    fieldname: ['mobile_no',"excel_sales_person_email","email_id"]
                },
                callback: function(response) {
                    if (response.message) {
                     const {mobile_no,excel_sales_person_email,email_id}=response.message
                       frm.set_value("excel_customer_mobile_no",mobile_no );
                       frm.set_value("excel_sales_person_mail",excel_sales_person_email );
                       frm.set_value("excel_customer_email",email_id );
                    }
                }
            });
		}
	}
})
```
