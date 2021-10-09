# Find SQL Linked Server Depdendencies

```SQL
SELECT 
    Distinct 
    referenced_Server_name As LinkedServerName,
    referenced_schema_name AS LinkedServerSchema,
    referenced_database_name AS LinkedServerDB,
    referenced_entity_name As LinkedServerTable,
    OBJECT_NAME (referencing_id) AS ObjectUsingLinkedServer
FROM sys.sql_expression_dependencies
JOIN sys.objects o on o.object_id = referencing_id
-- WHERE o.name = '<VIEW / SPROC NAME>' --UNCOMMENT TO FILTER ON SPECIFIC VIEW OR SPROC
```