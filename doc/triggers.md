# Использование триггеров в LiteSql


### Delete trigger

```sql
drop trigger insert_log;
```

### Create trigger
```sql
CREATE TRIGGER audit_log AFTER INSERT ON COMPANY
BEGIN
   INSERT INTO AUDIT(EMP_ID, ENTRY_DATE)  VALUES (new.ID, datetime('now'));
END;        
```


### Create trigger
```sql

CREATE TRIGGER insert_log AFTER INSERT ON links


BEGIN
    -- insert the new artist first
    INSERT INTO logs (title, datetime) VALUES(NEW.link, DATETIME('NOW'));
	-- INSERT INTO logs (title) VALUES("ok PSPS");
    
    -- use the artist id to insert a new album
    -- INSERT INTO Albums(Title, ArtistId) VALUES(NEW.AlbumTitle, last_insert_rowid());
END;
```

-- Update trigger

```sql
CREATE TRIGGER log_contact   AFTER UPDATE ON links
     --  WHEN old.phone <> new.phone
     --    OR old.email <> new.email
BEGIN
      INSERT INTO logs (title, datetime) VALUES(NEW.link, OLD.link);
END
```