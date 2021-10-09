# Get Backup or Restore Status

Execute the following query to get the current backup or restore status with estimated completion time.

```sql
SELECT session_id AS SPID,
    command,
    a.TEXT AS Query,
    start_time,
    percent_complete,
    dateadd(second, estimated_completion_time / 1000, getdate()) AS estimated_completion_time
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) a
WHERE r.command IN ('BACKUP DATABASE', 'RESTORE DATABASE')
```