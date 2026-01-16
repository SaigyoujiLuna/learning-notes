```sql
START TRANSACTION;

INSERT INTO new_data(id, data) SELECT id, data FROM old_data;

DROP TABLE old_data;

# 如果audit_log插入失败，事务回滚无法回滚drop table
INSERT INTO audit_log(info, dt) VALUES ("move old data to new data", NOW());

COMMIT;
```