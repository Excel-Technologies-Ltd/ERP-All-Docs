# Excel Mongo Query

**Brand Wise Serial**
```javascript
db.getCollection('serial_no').aggregate(
  [
    {
      $match: {
        sales_invoice_name: { $exists: false },
        warehouse:{$in:["HO Current Stock - ETL","HO Reserve Stock - ETL","Ryans Reserved Stock - ETL","Work In Progress - ETL","HO Balaka Stock - ETL"]}
      }
    },
    {
      $lookup: {
        from: 'item',
        localField: 'item_code',
        foreignField: 'item_code',
        as: 'item_details'
      }
    },
    { $unwind: { path: '$item_details' } },
    {
      $match: { 'item_details.brand': {$in:["TP-LINK","TP-LINK VIGI","TP-LINK TAPO","MERCUSYS"]} }
    },
    {
      $project: {
        serial_no: 1,
        item_code: 1,
        item_name: 1,
        brand: '$item_details.brand',
        warehouse: 1,
        purchase_on: '$warranty.purchasedOn'
      }
    }
  ],
  { maxTimeMS: 60000, allowDiskUse: true }
);
```
**Latest serial History**
```javascript
db.getCollection('serial_no').aggregate(
  [
    {
      $match: {
        warehouse: 'Work In Progress - ETL'
      }
    },
    {
      $lookup: {
        from: 'serial_no_history',
        let: { serial: '$serial_no' },
        pipeline: [
          {
            $match: {
              $expr: {
                $eq: ['$serial_no', '$$serial']
              }
            }
          },
          { $sort: { eventDate: -1 } },
          { $limit: 1 }
        ],
        as: 'latest_history'
      }
    },
    {
      $unwind: {
        path: '$latest_history',
        preserveNullAndEmptyArrays: true
      }
    },
    {
      $project: {
        item_code: 1,
        item_name: 1,
        serial_no: 1,
        document: '$latest_history.document_no'
      }
    }
  ],
  { maxTimeMS: 60000, allowDiskUse: true }
);
```
```javascript
db.getCollection('serial_no').aggregate(
  [
    {
      $match: {
        sales_invoice_name: { $exists: true },
        warehouse: {
          $in: [
            'HO Current Stock - ETL',
            'HO Reserve Stock - ETL',
            'Ryans Reserved Stock - ETL'
          ]
        },
        'warranty.soldOn': {
          $gte: ISODate(
            '2025-08-01T00:00:00.000Z'
          ),
          $lte: ISODate(
            '2025-08-15T23:59:59.999Z'
          )
        }
      }
    },
    {
      $lookup: {
        from: 'item',
        localField: 'item_code',
        foreignField: 'item_code',
        as: 'item_details'
      }
    },
    { $unwind: { path: '$item_details' } },
    {
      $match: {
        'item_details.brand': {
          $in: [
            'TP-LINK',
            'TP-LINK VIGI',
            'TP-LINK TAPO'
          ]
        }
      }
    },
    {
      $project: {
        serial_no: 1,
        item_code: 1,
        item_name: 1,
        brand: '$item_details.brand',
        warehouse: 1,
        sold_on: '$warranty.soldOn'
      }
    }
  ],
  { maxTimeMS: 60000, allowDiskUse: true }
);
```
```javascript
db.getCollection('stock_entry').aggregate(
  [
    {
      $match: {
        stock_entry_type: 'Material Transfer',
        posting_date: {
          $in: [
            '2025-8-1',
            '2025-8-2',
            '2025-8-3',
            '2025-8-4',
            '2025-8-5',
            '2025-8-6',
            '2025-8-7',
            '2025-8-8',
            '2025-8-9',
            '2025-8-10',
            '2025-8-11',
            '2025-8-12',
            '2025-8-13',
            '2025-8-14',
            '2025-8-15'
          ]
        },
        default_s_warehouse: {
          $in: [
            'HO Reserve Stock - ETL',
            'HO Balaka Stock - ETL',
            'HO Current Stock - ETL',
            'Ryans Reserved Stock - ETL'
          ]
        }
      }
    },
    { $unwind: { path: '$items' } },
    { $match: { 'items.has_serial_no': 1 } },
    { $unwind: { path: '$items.serial_no' } },
    {
      $project: {
        item_code: '$items.item_code',
        item_name: '$items.item_name',
        territory: 1,
        serial_no: '$items.serial_no',
        posting_date: 1,
        s_warehouse: '$items.s_warehouse',
        t_warehouse: '$items.t_warehouse',
        stock_id: 1
      }
    }
  ],
  { maxTimeMS: 60000, allowDiskUse: true }
);
```




