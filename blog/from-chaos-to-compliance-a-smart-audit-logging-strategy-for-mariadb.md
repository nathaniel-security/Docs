# From Chaos to Compliance A Smart Audit Logging Strategy for MariaDB

<figure><img src="../.gitbook/assets/ChatGPT Image Jun 8, 2025, 01_22_22 PM.png" alt=""><figcaption></figcaption></figure>

When working with production-grade systems, tracking changes to your data tables becomes crucial. In this post, we’ll walk through how to build a **self-maintaining audit log system in MariaDB** that captures every `INSERT`, `UPDATE`, and `DELETE`—complete with who did it, when, and what changed.

***

### 🎯 Goals of This System

* Automatically log `INSERT`, `UPDATE`, and `DELETE` actions.
* Record the table name, action type, primary key, and full row data.
* Include the current user (optional, via a session variable `@app_user`).
* Automatically detect and audit **new tables**.
* Automatically remove tracking for **deleted tables**.
* Use predictable trigger names (prefixed with `audit_`) for easy cleanup.

***

### 🏗 Step 1: Create the Audit Tables

#### `audit_log`

```sql
CREATE TABLE IF NOT EXISTS audit_log (  
    id INT AUTO_INCREMENT PRIMARY KEY,  
    table_name VARCHAR(255),  
    action_type ENUM('INSERT', 'UPDATE', 'DELETE'),  
    primary_key_value VARCHAR(255),  
    old_data TEXT,  
    new_data TEXT,  
    username VARCHAR(255) DEFAULT NULL,  
    action_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP  
);
```

#### `audit_trigger_tracker`

```sql
  CREATE TABLE IF NOT EXISTS audit_trigger_tracker (  
    table_name VARCHAR(255) PRIMARY KEY,  
    trigger_created BOOLEAN DEFAULT FALSE,  
    last_checked TIMESTAMP DEFAULT CURRENT_TIMESTAMP    
);
```

***

### 🔄 Step 2: Auto-Generate Triggers for New Tables

```sql
  
DELIMITER $$  
  
DROP PROCEDURE IF EXISTS generate_audit_triggers $$  
CREATE PROCEDURE generate_audit_triggers()  
BEGIN  
    DECLARE done INT DEFAULT FALSE;  
    DECLARE tbl_name VARCHAR(255);  
    DECLARE pk_column VARCHAR(255);  
    DECLARE old_json TEXT;  
    DECLARE new_json TEXT;  
  
    DECLARE cur CURSOR FOR  
        SELECT table_name  
        FROM information_schema.tables  
        WHERE table_schema = 'khoj'  
          AND table_name NOT IN (SELECT table_name FROM audit_trigger_tracker)  
          AND table_name NOT IN ('audit_log', 'audit_trigger_tracker');  
  
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;  
  
    OPEN cur;  
  
    read_loop: LOOP  
        FETCH cur INTO tbl_name;  
        IF done THEN  
            LEAVE read_loop;  
        END IF;  
  
        SELECT COLUMN_NAME INTO pk_column  
        FROM information_schema.columns  
        WHERE table_schema = 'khoj'  
          AND table_name = tbl_name  
          AND COLUMN_KEY = 'PRI'  
        LIMIT 1;  
  
        SELECT GROUP_CONCAT(CONCAT('"', COLUMN_NAME, '", OLD.', COLUMN_NAME) SEPARATOR ', ')  
        INTO old_json  
        FROM information_schema.columns  
        WHERE table_schema = 'khoj' AND table_name = tbl_name;  
  
        SELECT GROUP_CONCAT(CONCAT('"', COLUMN_NAME, '", NEW.', COLUMN_NAME) SEPARATOR ', ')  
        INTO new_json  
        FROM information_schema.columns  
        WHERE table_schema = 'khoj' AND table_name = tbl_name;  
  
        SET @sql_insert = CONCAT('  
            CREATE TRIGGER `audit_', tbl_name, '_after_insert`  
            AFTER INSERT ON `', tbl_name, '`  
            FOR EACH ROW            BEGIN                INSERT INTO audit_log (table_name, action_type, primary_key_value, new_data, username)                VALUES ("', tbl_name, '", "INSERT", NEW.', pk_column, ',  
                        JSON_OBJECT(', new_json, '), @app_user);  
            END;');  
  
        SET @sql_update = CONCAT('  
            CREATE TRIGGER `audit_', tbl_name, '_after_update`  
            AFTER UPDATE ON `', tbl_name, '`  
            FOR EACH ROW            BEGIN                INSERT INTO audit_log (table_name, action_type, primary_key_value, old_data, new_data, username)                VALUES ("', tbl_name, '", "UPDATE", NEW.', pk_column, ',  
                        JSON_OBJECT(', old_json, '),  
                        JSON_OBJECT(', new_json, '),  
                        @app_user);            END;');  
  
        SET @sql_delete = CONCAT('  
            CREATE TRIGGER `audit_', tbl_name, '_after_delete`  
            AFTER DELETE ON `', tbl_name, '`  
            FOR EACH ROW            BEGIN                INSERT INTO audit_log (table_name, action_type, primary_key_value, old_data, username)                VALUES ("', tbl_name, '", "DELETE", OLD.', pk_column, ',  
                        JSON_OBJECT(', old_json, '), @app_user);  
            END;');  
  
        PREPARE stmt FROM @sql_insert; EXECUTE stmt; DEALLOCATE PREPARE stmt;  
        PREPARE stmt FROM @sql_update; EXECUTE stmt; DEALLOCATE PREPARE stmt;  
        PREPARE stmt FROM @sql_delete; EXECUTE stmt; DEALLOCATE PREPARE stmt;  
  
        INSERT INTO audit_trigger_tracker (table_name, trigger_created)  
        VALUES (tbl_name, TRUE);  
    END LOOP;  
  
    CLOSE cur;  
END $$  
DELIMITER ;

CALL generate_audit_triggers();
```

***

### 🧼 Step 3: Clean Up Dropped Tables (Triggers + Tracker)

```sql
DELIMITER $$  
DROP PROCEDURE IF EXISTS drop_missing_table_audit_triggers $$  
CREATE PROCEDURE drop_missing_table_audit_triggers()  
BEGIN  
  DECLARE done INT DEFAULT FALSE;  
  DECLARE tbl VARCHAR(255);  
  DECLARE cur CURSOR FOR  
    SELECT table_name  
    FROM audit_trigger_tracker  
    WHERE table_name NOT IN (  
      SELECT table_name FROM information_schema.tables WHERE table_schema = 'khoj'  
    );  
  
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;  
  
  OPEN cur;  
  
  loop_triggers: LOOP  
    FETCH cur INTO tbl;  
    IF done THEN  
      LEAVE loop_triggers;  
    END IF;  
  
    SET @sql1 = CONCAT('DROP TRIGGER IF EXISTS `audit_', tbl, '_after_insert`;');  
    SET @sql2 = CONCAT('DROP TRIGGER IF EXISTS `audit_', tbl, '_after_update`;');  
    SET @sql3 = CONCAT('DROP TRIGGER IF EXISTS `audit_', tbl, '_after_delete`;');  
  
    PREPARE stmt1 FROM @sql1; EXECUTE stmt1; DEALLOCATE PREPARE stmt1;  
    PREPARE stmt2 FROM @sql2; EXECUTE stmt2; DEALLOCATE PREPARE stmt2;  
    PREPARE stmt3 FROM @sql3; EXECUTE stmt3; DEALLOCATE PREPARE stmt3;  
  
    DELETE FROM audit_trigger_tracker WHERE table_name = tbl;  
  END LOOP;  
  
  CLOSE cur;  
END $$  
DELIMITER ;  
  

```

#### Cron

```
CALL drop_missing_table_audit_triggers();  
CALL generate_audit_triggers();
```

***

### 🧨 Nuking the Entire Audit Setup

To completely remove the audit system—including all triggers and supporting tables—use:

```sql
DELIMITER $$  
  
DROP PROCEDURE IF EXISTS drop_all_audit_triggers $$  
CREATE PROCEDURE drop_all_audit_triggers()  
BEGIN  
  DECLARE done INT DEFAULT FALSE;  
  DECLARE trg_name VARCHAR(255);  
  DECLARE cur CURSOR FOR  
    SELECT TRIGGER_NAME  
    FROM information_schema.triggers  
    WHERE TRIGGER_SCHEMA = DATABASE()  
      AND TRIGGER_NAME LIKE 'audit\\_%';  
  
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;  
  
  OPEN cur;  
  
  trg_loop: LOOP  
    FETCH cur INTO trg_name;  
    IF done THEN  
      LEAVE trg_loop;  
    END IF;  
  
    SET @sql = CONCAT('DROP TRIGGER IF EXISTS `', trg_name, '`');  
    PREPARE stmt FROM @sql;  
    EXECUTE stmt;  
    DEALLOCATE PREPARE stmt;  
  END LOOP;  
  
  CLOSE cur;  
END$$  
  
DELIMITER ;  
  
-- 🔥 ExecuteCALL drop_all_audit_triggers();  
  
-- 🧹 Clean upDROP PROCEDURE drop_all_audit_triggers;  
drop procedure drop_missing_table_audit_triggers;  
drop procedure generate_audit_triggers;  
  
DROP TABLE IF EXISTS audit_log;  
DROP TABLE IF EXISTS audit_trigger_tracker;
```

***

### 🤖 Automate With Cron

Create a bash script like this:

```bash
#!/bin/bash
mysql -u root -p"yourpassword" -h 127.0.0.1 -P 3306 -D khoj -e "
CALL drop_missing_table_audit_triggers();
CALL generate_audit_triggers();"
```

Then schedule it:

```bash
*/5 * * * * /path/to/audit_trigger_update.sh
```

***

### ✅ Conclusion

You now have a fully autonomous audit logging system:

* Self-healing on schema changes
* Row-level forensic history
* Optional user tracking with `@app_user`
* Clean rollback mechanism
