# Appservice Vnet Integration

## Gateway Integration

When using gateway integration if you need to peer to another vnet the following must be configured.

**!! Any changes to routes in the vnet integration may require a Network Sync performed on the App plan !!**

### Source Vnet
Ensure that the peering has the following set
* allowVirtualNetworkAccess: true
* allowForwardedTraffic: true
* allowGatewayTransit: true

### Destination Vnet
Ensure that the peering has the following set
* allowVirtualNetworkAccess: true
* allowForwardedTraffic: true
* useRemoteGateways: true

### App Plan Vnet Integration
Add the subnet range to the routes. Either manually within the networking on the App Plan or by using "Microsoft.Web/serverfarms/virtualNetworkConnections/routes" ARM type.

Example

```json
{
  "name": "string",
  "type": "Microsoft.Web/serverfarms/virtualNetworkConnections/routes",
  "apiVersion": "2019-08-01",
  "kind": "string",
  "properties": {
    "startAddress": "string",
    "endAddress": "string",
    "routeType": "string"
  }
}
```
