# Let's get Start

**Configure A records:**

- Add A record for portainer.example.com pointing the public IP
- Add A record for traefik.example.com pointing the public IP
- Add A record for erp.example.com pointing the public IP
- Add A record for rma.example.com pointing the public IP
- Add A record for warranty.example.com pointing the public IP

# Step 1

[ Setup Portainer](./portainer/portainer.md)

after successfully setup portainer
go to "portainer.example.com"

# Step 2

[ Create Erpnext Stack and Restore Mariadb Database](./yaml/erpnext.md)

# Step 3

[ Create Mongodb-Replica Stack and Restore Mongo Database](./yaml/mongo.md)

# Step 4

[ Create RMA-Server Stack](./yaml/rma-server.md)

# Once done everything we need to configure some basic changes

###

- Delete all Webhook from
- Update RMA `server_settings` from mongodb-master with your credentials

**Run the command, change the variables as required. Before running this, generate the serviceAccount ApiSecret again for the account and replace it. As we are restoring database, frontend/backend client id doesn't change, service account already exists, check the erpnext site name and then run this.**

```bash
db.server_settings.updateOne({},{
	$set:{
        appURL: 'https://drill-rma.arcapps.org',
        warrantyAppURL: 'https://drill-warranty.arcapps.org',
        frontendClientId: '1cae8c4f76',
        backendClientId: 'fe6df0a17b',
        serviceAccountUser: 'bot@excelbd.com',
        serviceAccountSecret: 'VTtVNLTvqi0K9tl',
        authServerURL: 'https://drill-erp.arcapps.org',
        serviceAccountApiKey: '403dfa271ac2021',
        serviceAccountApiSecret: '7ba6c594ec95a0c',
        profileURL: 'https://drill-erp.arcapps.org/api/method/frappe.integrations.oauth2.openid_profile',
        authorizationURL: 'https://drill-erp.arcapps.org/api/method/frappe.integrations.oauth2.authorize',
        tokenURL: 'https://drill-erp.arcapps.org/api/method/frappe.integrations.oauth2.get_token',
        revocationURL: 'https://drill-erp.arcapps.org/api/method/frappe.integrations.oauth2.revoke_token',
        scope: [ 'all', 'openid' ],
        webhookApiKey: 'b9fcc4c8c1be9b6681c6ec20e2c9935ef2f8c450b9728f7c7e00bbfc38076a00f3740cdbad884c97d64db24389e15dae0156590ee840ffa54d3348fd827d79d0',
        timeZone: 'Asia/Dhaka',
        defaultCompany: 'Excel Technologies Ltd.',
        sellingPriceList: 'Standard Selling',
        debtorAccount: '10203 - Accounts Receivable - ETL',
        transferWarehouse: 'Work In Progress - ETL',
        validateStock: true,
        footerImageURL: 'https://drill-erp.arcapps.org/files/SINV-Footer.jpg',
        footerWidth: 530,
        headerImageURL: 'https://drill-erp.arcapps.org/files/ETL-header.jpg',
        headerWidth: 500,
        brand: { faviconURL: 'https://drill-erp.arcapps.org/files/Fav-Icon.png' },
        backdatedInvoices: true,
        posAppURL: 'https://pos.excelbd.com',
        backdatedInvoicesForDays: null,
        posProfile: 'Service Center'
      }
})
```

### Setup Webhooks

Set the all Webhook which we deleted previously from ERP With Send a `POST` request by RESTER/Postman

```tsx
{
  "Authorization": "Bearer <system manager token>"
}

```

This will create Webhooks in ERPNext synchronously. If it fails, delete the created webhooks from ERPNext and make this request again

### Congratulations!!! You're good to go.... Everything completed successfully

### If you encounter an error or don't understand how to establish a connection between RMA and ERPNext, please watch this video

[Video Link](https://www.youtube.com/watch?v=Vk1iF7Lam3g)
