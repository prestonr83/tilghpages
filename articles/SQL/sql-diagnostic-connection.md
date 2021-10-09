# SQL Diagnostic Connection
When troubleshooting SQL if the server is unresponsive and you are unable to connect via SSMS then you can use a Diagnostic connection.

Microsoft Documentation Link - [Diagnostic Connection for Database Administrators](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/diagnostic-connection-for-database-administrators?view=sql-server-ver15)

Steps for connection
* Open SSMS
* Close the Login Window
* Click File > New > Database Engine Query
* append `admin:` to the server name and login with valid SA level credentials
![example gif](../Media/ssmsDAC.gif)