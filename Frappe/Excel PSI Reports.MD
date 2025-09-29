```
SELECT ti.item_code AS `Item Code`,
    ti.item_name AS `Item Name`,
    ti.item_group AS `Item Group`,
    ti.brand AS `Brand`,
    COALESCE(psi.fob, 0) AS `FOB`,
    COALESCE(tip.price_list_rate, 0) AS `Stock Unit Price`,
    COALESCE(tb.actual_qty, 0) AS `Current Stock Qty`,
    COALESCE(psi.pipeline, 0) AS `Pipeline`,
    COALESCE(psi.under_production, 0) AS `Under Production`,
    (
        COALESCE(tb.actual_qty, 0) + COALESCE(psi.pipeline, 0) + COALESCE(psi.under_production, 0)
    ) AS `Total Stock (Incl. Pipeline)`,
    COALESCE(psi.proposed_new_order_qty, 0) AS `Proposed New Order Qty`,
    COALESCE(psi.suggested__order, 0) AS `Suggested Order`,
    (
        CASE
            WHEN COALESCE(combined_sales.`Average Sales`, 0) <> 0 THEN (
                (
                    COALESCE(tb.actual_qty, 0) + COALESCE(psi.pipeline, 0) + COALESCE(psi.under_production, 0)
                ) / combined_sales.`Average Sales`
            )
            ELSE 0
        END
    ) AS `Stock Age (Months)`,
    COALESCE(combined_sales.`Average Sales`, 0) AS `Average Sales`,
    COALESCE(combined_sales.`Max Sales`, 0) AS `Max Sales`,
    COALESCE(combined_sales.`Current Month`, 0) AS `Current Month`,
    COALESCE(combined_sales.`Last Month`, 0) AS `Last Month`,
    COALESCE(combined_sales.`2 Months Ago`, 0) AS `2 Months Ago`,
    COALESCE(combined_sales.`3 Months Ago`, 0) AS `3 Months Ago`,
    COALESCE(combined_sales.`4 Months Ago`, 0) AS `4 Months Ago`,
    COALESCE(combined_sales.`5 Months Ago`, 0) AS `5 Months Ago`,
    COALESCE(combined_sales.`6 Months Ago`, 0) AS `6 Months Ago`,
    COALESCE(combined_sales.`7 Months Ago`, 0) AS `7 Months Ago`,
    COALESCE(combined_sales.`8 Months Ago`, 0) AS `8 Months Ago`,
    COALESCE(combined_sales.`9 Months Ago`, 0) AS `9 Months Ago`,
    COALESCE(combined_sales.`10 Months Ago`, 0) AS `10 Months Ago`,
    COALESCE(combined_sales.`11 Months Ago`, 0) AS `11 Months Ago`
FROM `tabItem` AS ti
    LEFT JOIN (
        SELECT tb.item_code AS `item_code`,
            SUM(tb.actual_qty) AS `actual_qty`
        FROM `tabBin` AS tb
        WHERE (
                tb.warehouse NOT LIKE '%RMA%'
                AND tb.warehouse NOT LIKE '%Foreign%'
                AND tb.warehouse NOT LIKE '%CSP Current%'
                AND tb.warehouse NOT LIKE '%Bad%'
            )
        GROUP BY tb.item_code
    ) AS tb ON ti.item_code = tb.item_code
    LEFT JOIN (
        SELECT ip.item_code,
            ip.price_list_rate
        FROM `tabItem Price` AS ip
        WHERE ip.price_list = "Bottom Price"
            AND ip.selling = 1
    ) AS tip ON ti.item_code = tip.item_code
    LEFT JOIN (
        SELECT psi.item_code,
            psi.pipeline,
            psi.under_production,
            psi.proposed_new_order_qty,
            psi.suggested__order,
            psi.fob
        FROM `tabExcel PSI` AS psi
    ) AS psi ON ti.item_code = psi.item_code
    LEFT JOIN (
        WITH non_bundle_sales AS (
            SELECT `tabSales Invoice Item`.item_code AS `Item Code`,
                SUM(`tabSales Invoice Item`.stock_qty) AS `Quantity`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(CURRENT_DATE, '%Y-%m') THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `Current Month`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 1 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `Last Month`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 2 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `2 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 3 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `3 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 4 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `4 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 5 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `5 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `6 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 7 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `7 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 8 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `8 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 9 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `9 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 10 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `10 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 11 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty
                        ELSE 0
                    END
                ) AS `11 Months Ago`
            FROM `tabSales Invoice Item`
                JOIN (
                    SELECT ti.is_stock_item,
                        ti.name
                    FROM `tabItem` AS ti
                        LEFT JOIN `tabProduct Bundle Item` AS pbi ON ti.name = pbi.parent
                    WHERE ti.is_stock_item = 1
                        AND pbi.parent IS NULL
                ) AS nbp ON `tabSales Invoice Item`.item_code = nbp.name
                JOIN `tabSales Invoice` ON `tabSales Invoice`.name = `tabSales Invoice Item`.parent
            WHERE `tabSales Invoice`.posting_date >= DATE_SUB(CURRENT_DATE, INTERVAL 12 MONTH)
                AND `tabSales Invoice`.docstatus = 1
            GROUP BY `tabSales Invoice Item`.item_code
        ),
        bundle_sales AS (
            SELECT tbn.bundle_single_item_code AS `Item Code`,
                SUM(
                    `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                ) AS `Quantity`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(CURRENT_DATE, '%Y-%m') THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `Current Month`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 1 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `Last Month`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 2 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `2 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 3 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `3 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 4 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `4 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 5 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `5 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `6 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 7 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `7 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 8 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `8 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 9 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `9 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 10 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `10 Months Ago`,
                SUM(
                    CASE
                        WHEN DATE_FORMAT(`tabSales Invoice`.posting_date, '%Y-%m') = DATE_FORMAT(
                            DATE_SUB(CURRENT_DATE, INTERVAL 11 MONTH),
                            '%Y-%m'
                        ) THEN `tabSales Invoice Item`.stock_qty * tbn.bundle_qty
                        ELSE 0
                    END
                ) AS `11 Months Ago`
            FROM `tabSales Invoice Item`
                JOIN `tabSales Invoice` ON `tabSales Invoice`.name = `tabSales Invoice Item`.parent
                LEFT JOIN (
                    SELECT ti.item_code AS `bundle_item_code`,
                        tbp.bundle_patent_code,
                        tbp.bundle_qty,
                        tbp.bundle_single_item_code,
                        ti.is_stock_item AS `is_stock_item`
                    FROM `tabItem` AS ti
                        LEFT JOIN (
                            SELECT `tabProduct Bundle Item`.parent AS `bundle_patent_code`,
                                `tabProduct Bundle Item`.item_code AS `bundle_single_item_code`,
                                `tabProduct Bundle Item`.qty AS `bundle_qty`
                            FROM `tabProduct Bundle Item`
                            WHERE `tabProduct Bundle Item`.parentfield = 'items'
                                AND `tabProduct Bundle Item`.parenttype = 'Product Bundle'
                        ) AS tbp ON ti.item_code = tbp.bundle_patent_code
                    WHERE ti.is_stock_item = 0
                ) AS tbn ON `tabSales Invoice Item`.item_code = tbn.bundle_item_code
            WHERE `tabSales Invoice`.docstatus = 1
                AND tbn.is_stock_item = 0
                AND `tabSales Invoice`.posting_date >= DATE_SUB(CURRENT_DATE, INTERVAL 12 MONTH)
            GROUP BY tbn.bundle_single_item_code
        )
        SELECT `Item Code` as `item_code`,
            SUM(COALESCE(`Current Month`, 0)) AS `Current Month`,
            SUM(COALESCE(`Last Month`, 0)) AS `Last Month`,
            SUM(COALESCE(`2 Months Ago`, 0)) AS `2 Months Ago`,
            SUM(COALESCE(`3 Months Ago`, 0)) AS `3 Months Ago`,
            SUM(COALESCE(`4 Months Ago`, 0)) AS `4 Months Ago`,
            SUM(COALESCE(`5 Months Ago`, 0)) AS `5 Months Ago`,
            SUM(COALESCE(`6 Months Ago`, 0)) AS `6 Months Ago`,
            SUM(COALESCE(`7 Months Ago`, 0)) AS `7 Months Ago`,
            SUM(COALESCE(`8 Months Ago`, 0)) AS `8 Months Ago`,
            SUM(COALESCE(`9 Months Ago`, 0)) AS `9 Months Ago`,
            SUM(COALESCE(`10 Months Ago`, 0)) AS `10 Months Ago`,
            SUM(COALESCE(`11 Months Ago`, 0)) AS `11 Months Ago`,
            GREATEST(
                SUM(COALESCE(`Current Month`, 0)),
                SUM(COALESCE(`Last Month`, 0)),
                SUM(COALESCE(`2 Months Ago`, 0)),
                SUM(COALESCE(`3 Months Ago`, 0)),
                SUM(COALESCE(`4 Months Ago`, 0)),
                SUM(COALESCE(`5 Months Ago`, 0)),
                SUM(COALESCE(`6 Months Ago`, 0)),
                SUM(COALESCE(`7 Months Ago`, 0)),
                SUM(COALESCE(`8 Months Ago`, 0)),
                SUM(COALESCE(`9 Months Ago`, 0)),
                SUM(COALESCE(`10 Months Ago`, 0)),
                SUM(COALESCE(`11 Months Ago`, 0))
            ) AS `Max Sales`,
            (
                SUM(COALESCE(`Current Month`, 0)) + SUM(COALESCE(`Last Month`, 0)) + SUM(COALESCE(`2 Months Ago`, 0)) + SUM(COALESCE(`3 Months Ago`, 0)) + SUM(COALESCE(`4 Months Ago`, 0)) + SUM(COALESCE(`5 Months Ago`, 0)) + SUM(COALESCE(`6 Months Ago`, 0)) + SUM(COALESCE(`7 Months Ago`, 0)) + SUM(COALESCE(`8 Months Ago`, 0)) + SUM(COALESCE(`9 Months Ago`, 0)) + SUM(COALESCE(`10 Months Ago`, 0)) + SUM(COALESCE(`11 Months Ago`, 0))
            ) / 12 AS `Average Sales`
        FROM (
                SELECT *
                FROM non_bundle_sales
                UNION ALL
                SELECT *
                FROM bundle_sales
            ) AS combined_sales
        GROUP BY `Item Code`
    ) combined_sales ON combined_sales.item_code = ti.item_code
WHERE ti.is_stock_item = 1
GROUP BY ti.item_code


```
