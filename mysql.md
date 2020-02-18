user, grant

```sql
CREATE USER app@'%' IDENTIFIED BY '<password>';
GRANT SELECT, INSERT, UPDATE, DELETE ON `<schema>`.* TO app@'%';

CREATE USER migration@'%' IDENTIFIED BY '<password>';
GRANT ALTER, CREATE, DROP, INDEX, INSERT, UPDATE, DELETE, SELECT ON <schema>.* TO migration@'%';

CREATE USER admin@'%' IDENTIFIED BY '<password>';
GRANT ALL ON `<schema>`.* TO admin@'%';
```
