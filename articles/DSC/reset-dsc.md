# Reset DSC Local Configuration Manager

```powershell
[DscLocalConfigurationManager()]
Configuration ResetLCM {
    Node localhost {
        Settings {
            RebootNodeIfNeeded = $True
            ConfigurationMode = 'ApplyAndMonitor'
            RefreshMode = 'Push'
            ActionAfterReboot = 'ContinueConfiguration'
        }
    }
}

ResetLCM -out c:\$env:temp\resetLCM
Set-DscLocalConfigurationManager -Path C:\$env:temp\resetLCM -Verbose
```