# Add Login Mapping to Linked Server

## Mapped Login to Remote SQL Login
```SQL
USE [master]
GO
EXEC master.dbo.sp_addlinkedsrvlogin @rmtsrvname = N'<SERVER NAME>', @locallogin = N'<LOCAL LOGIN>', @useself = N'False', @rmtuser = N'<REMOTE LOGIN>', @rmtpassword = N'<REMOTE PASSWORD>'
```

## User Impersonation
```SQL
USE [master]
GO
EXEC master.dbo.sp_addlinkedsrvlogin @rmtsrvname = N'<SERVER NAME>', @locallogin = N'<LOCAL LOGIN>', @useself = N'True'
```