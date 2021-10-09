# List all Database filenames and locations

```sql
SELECT
    db.name AS DBName,
	mf.name AS FileName,
    type_desc AS FileType,
    Physical_Name AS Location
FROM
    sys.master_files mf
INNER JOIN 
    sys.databases db ON db.database_id = mf.database_id
```