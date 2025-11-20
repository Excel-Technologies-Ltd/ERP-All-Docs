# **Production ERP Troubleshooting Guide**

This guide outlines the standard operating procedure for updating, scaling, and optimizing the production ERP system running in Docker (Portainer) with master-slave database replication.

---

## **Step 1: Update Containers in Portainer**
1. Login to **Portainer**.
2. Navigate to: **Stacks → frappe-bench-v14**.
3. Select **all containers**.
4. Click **Update** to ensure images and configurations are refreshed.

---

## **Step 2: Scale Services**
On the same stack:

| Service Name       | Scale Count |
|--------------------|-------------|
| backend            | 2           |
| queue-default      | 5           |
| queue-long         | 5           |
| queue-short        | 5           |

Make sure the scaling operation finishes successfully before moving to the next steps.

---

## **Step 3: Enable Scheduler**
1. Open the terminal of any **backend container**.
2. Run the following command:

```bash
bench --site all enable-scheduler
```

This ensures background jobs and scheduled events run properly.

---

## **Step 4: Update Database Settings**
Perform on both **MariaDB master and slave instances**.

1. Login to the database:

```bash
mysql -u root -p yourpassword
```

2. Set the buffer pool size (adjust value based on server RAM):

```sql
SET GLOBAL innodb_buffer_pool_size=(40 * 1024 * 1024 * 1024);
```

> Recommended buffer pool size: **50–70% of total RAM**.

3. Verify the resize status:

```sql
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_resize_status';
```

You should see a success message if it applied correctly.

---

## Notes & Best Practices
- Always run scaling changes during low traffic hours.
- Changing buffer size requires MariaDB 10.6+ and dynamic resizing enabled.
- Monitor logs using:

```bash
docker logs <container-name> -f
```

- Validate scheduler logs via:

```bash
bench doctor
```

---

### Completion Check
| Task | Status |
|------|--------|
| All containers updated | ☐ |
| Scaling applied | ☐ |
| Scheduler enabled | ☐ |
| Database tuning applied | ☐ |
| Logs verified | ☐ |

---
