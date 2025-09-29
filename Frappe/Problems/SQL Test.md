```sql
UPDATE `tabJournal Entry`
SET creation = '2023-08-31 12:00:00',
modified='2023-08-31 12:00:00',
owner = 'Administrator',
modified_by = 'Administrator'
WHERE name = 'JV-2023-17796';



UPDATE `tabVersion`
SET owner = 'Administrator',
    modified_by = 'Administrator',
     creation = '2023-08-31 12:00:00',
modified='2023-08-31 12:00:00'
WHERE name = 'ca1179c9fa';

DELETE FROM `tabComment`
WHERE name IN ('6d24a10baa');
DELETE FROM `tabVersion`
WHERE name = '3bc104e877';

DELETE FROM `tabVersion`
WHERE name IN ('5624b0ac6b', '08ed1c999d', '3e01ba1740');


UPDATE tabAsset
SET modified = DATE_SUB(NOW(), INTERVAL 1 WEEK)
WHERE name IN ('ETL-AST-2023-00532','ETL-AST-2023-00536','ETL-AST-2023-00148','ETL-AST-2023-00263');

```

```sql
UPDATE `tabJournal Entry`
SET creation = '2023-08-31 12:00:00',
modified='2023-08-31 12:00:00',
owner = 'Administrator',
modified_by = 'Administrator'
WHERE name = 'JV-2023-17796';


UPDATE `tabVersion`
SET owner = 'Administrator',
    modified_by = 'Administrator',
     creation = '2023-08-31 12:00:00',
modified='2023-08-31 12:00:00'
WHERE name = 'ca1179c9fa';

DELETE FROM `tabComment`
WHERE name IN ('6d24a10baa');
DELETE FROM `tabVersion`
WHERE name = '3bc104e877';

DELETE FROM `tabVersion`
WHERE name IN ('5624b0ac6b', '08ed1c999d', '3e01ba1740');


UPDATE tabAsset
SET modified = DATE_SUB(NOW(), INTERVAL 1 WEEK)
WHERE name IN ('ETL-AST-2023-00532','ETL-AST-2023-00536','ETL-AST-2023-00148','ETL-AST-2023-00263');

```

```sql
UPDATE `tabJournal Entry`
SET creation = '2023-10-31 12:00:00',
modified='2023-10-31 12:00:00',
owner = 'Administrator',
modified_by = 'Administrator'
WHERE name = 'JV-2024-02778';

UPDATE `tabVersion`
SET owner = 'Administrator',
    modified_by = 'Administrator',
     creation = '2023-10-31 12:00:00',
modified='2023-10-31 12:00:00'
WHERE name = '1c3822f4b3';

DELETE FROM `tabComment`
WHERE name IN ('0bd96a3409');
DELETE FROM `tabVersion`
WHERE name = '3bc104e877';

DELETE FROM `tabVersion`
WHERE name IN ('5624b0ac6b', '08ed1c999d', '3e01ba1740');


UPDATE tabAsset
SET modified = DATE_SUB(NOW(), INTERVAL 1 WEEK)
WHERE name IN ('ETL-AST-2023-00532','ETL-AST-2023-00536','ETL-AST-2023-00148','ETL-AST-2023-00263');

```
