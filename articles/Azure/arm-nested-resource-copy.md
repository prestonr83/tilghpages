# Copying nested resources in ARM templates

In order to copy nested resources in a template you need to move them out to be root resources. In order to do that change the name so it contains the name of the root resource / child resource. Also change the type to full type.

The following example would fail validation.

```json
{
    "apiVersion": "2016-03-01",
    "name": "[parameters('webAppName')]",
    "type": "Microsoft.Web/sites",
    "location": "[resourceGroup().location]",
    "properties": {
        "name": "[parameters('webAppName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverFarms',variables('appServicePlanName'))]"
    },
    "dependsOn": [
        "[concat('Microsoft.Web/serverFarms/',variables('appServicePlanName'))]"
    ],
    "resources": [
        {
            "type":"hostnameBindings",
            "name":"[parameters('customHostname')[copyindex()]]",
            "copy": {
                "name":"bindings",
                "count":2
            },
            "apiVersion":"2016-03-01",
            "location":"[resourceGroup().location]",
            "properties":{
                "sslState":"SniEnabled",
                "thumbprint":"[reference(resourceId('Microsoft.Web/certificates', variables('certificateName'))).Thumbprint]"
            },
            "dependsOn": [
                "[concat('Microsoft.Web/certificates/',variables('certificateName'))]"
            ]
        }
    ]
}
```

Instead should be done like so

```json
{
    "apiVersion": "2016-03-01",
    "name": "[parameters('webAppName')]",
    "type": "Microsoft.Web/sites",
    "location": "[resourceGroup().location]",
    "properties": {
        "name": "[parameters('webAppName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverFarms',variables('appServicePlanName'))]"
    },
    "dependsOn": [
        "[concat('Microsoft.Web/serverFarms/',variables('appServicePlanName'))]"
    ]
},
{
    "type":"Microsoft.Web/sites/hostnameBindings",
    "name":"[concat(parameters('webAppName'), '/', parameters('customHostname')[copyIndex()])]",
    "copy": {
        "name":"bindings",
        "count":2
    },
    "apiVersion":"2016-03-01",
    "location":"[resourceGroup().location]",
    "properties":{
        "sslState":"SniEnabled",
        "thumbprint":"[reference(resourceId('Microsoft.Web/certificates', variables('certificateName'))).Thumbprint]"
    },
    "dependsOn": [
        "[concat('Microsoft.Web/certificates/',variables('certificateName'))]"
    ]
}
```


