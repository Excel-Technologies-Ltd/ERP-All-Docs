# Database Connection Issue

**If you're in as root, reset the user password**
```typescript
ALTER USER '_465d519e3a50ea29'@'%' IDENTIFIED BY 'WQCSvnUJxyy8fz2B';
FLUSH PRIVILEGES;
```

 **Also try granting all privileges just in case:**
 ```typescript
GRANT ALL PRIVILEGES ON _465d519e3a50ea29.* TO '_465d519e3a50ea29'@'%' IDENTIFIED BY 'WQCSvnUJxyy8fz2B';
FLUSH PRIVILEGES;
```
