# Adding SQL Users and Logins

## Sql Auth Master Login and User

```sql
-- create SQL auth login from master 
CREATE LOGIN test 
WITH PASSWORD = 'SuperSecret!' 

-- select your db in the dropdown and create a user mapped to a login 
CREATE USER [test] 
FOR LOGIN [test] 
WITH DEFAULT_SCHEMA = dbo; 
  
-- add user to role(s) in db 
ALTER ROLE db_datareader ADD MEMBER [test]; 
ALTER ROLE db_datawriter ADD MEMBER [test]; 
```

## Sql Auth Contained User
```sql
-- select your db in dropdown and create a contained user 
CREATE USER [test] 
WITH PASSWORD = 'SuperSecret!', 
DEFAULT_SCHEMA = dbo; 
  
-- add user to role(s) in db 
ALTER ROLE db_datareader ADD MEMBER [test]; 
ALTER ROLE db_datawriter ADD MEMBER [test]; 
```

## AAD Auth Contained User
```sql
-- add contained Azure AD user 
CREATE USER [name@domain.com] 
FROM EXTERNAL PROVIDER 
WITH DEFAULT_SCHEMA = dbo;  
  
-- add user to role(s) in db 
ALTER ROLE dbmanager ADD MEMBER [name@domain.com]; 
ALTER ROLE loginmanager ADD MEMBER [name@domain.com]; 
```

## AAD Auth Admin User
```sql
-- add contained Azure AD user 
CREATE USER [name@domain.com] 
FROM EXTERNAL PROVIDER 
WITH DEFAULT_SCHEMA = dbo;  
  
-- add user to role(s) in db 
ALTER ROLE dbmanager ADD MEMBER [name@domain.com]; 
ALTER ROLE loginmanager ADD MEMBER [name@domain.com]; 
```

## Azure Managed Identity
```sql
CREATE USER [<identity-name>] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [<identity-name>];
ALTER ROLE db_datawriter ADD MEMBER [<identity-name>];
ALTER ROLE db_ddladmin ADD MEMBER [<identity-name>];
```
*If using slots <identity-name> would be \<app-name\>/slots/\<slot-name\>*
