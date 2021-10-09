# Capture 4xx/5xx Status Codes using Invoke-WebRequest

By default IWR throws an error and does not return an object when a 4xx/5xx error is thrown. Use the following to capture those statuses and return them.

```powershell
try { $response = Invoke-WebRequest http://localhost/foo } catch {
      $_.Exception.Response.StatusCode.Value__}
```