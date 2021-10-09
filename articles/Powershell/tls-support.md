# Enable TLS 1.2 Support in Powershell script

```powershell
[Net.ServicePointManager]::SecurityProtocol = "tls12, tls11, tls"
```