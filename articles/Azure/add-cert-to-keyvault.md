# Add PFX Certificate to Azure Key Vault as Secret

```powershell
$pfxFile = "" #Full Path to PFX File.
$mypwd = ConvertTo-SecureString -String "" -Force -AsPlainText #Password to PFX File.
$vaultName = "" # Name of key vault.
$secretName = "" # Name of secret.

$exportable = [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]"Exportable"
$cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2("$pfxFile", `
                                                                                  $mypwd, ` # If no password remove this line.
                                                                                  $exportable)
$secretvalue = ConvertTo-SecureString ([System.Convert]::ToBase64String($cert.Export('Pfx'))) -AsPlainText -Force
Set-AzKeyVaultSecret -vaultName $vaultName -Name $secretName -SecretValue $secretvalue -ContentType "application/x-pkcs12"
```