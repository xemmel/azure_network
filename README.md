## Azure networking

## Table of Content

4. [Private Endpoints](#private-endpoints)


### Private Endpoints

#### ARM


##### Private DNS Zone
```json

  {
            "type": "Microsoft.Network/privateDnsZones",
            "apiVersion": "2018-09-01",
            "name": "[parameters('privateDnsZones_privatelink_azurewebsites_net_name')]",
            "location": "global",
            "properties": {
                "maxNumberOfRecordSets": 25000,
                "maxNumberOfVirtualNetworkLinks": 1000,
                "maxNumberOfVirtualNetworkLinksWithRegistration": 100,
                "numberOfRecordSets": 3,
                "numberOfVirtualNetworkLinks": 0,
                "numberOfVirtualNetworkLinksWithRegistration": 0,
                "provisioningState": "Succeeded"
            }
        }

```
##### Private DNS Zone (A)

```json

 {
            "type": "Microsoft.Network/privateDnsZones/A",
            "apiVersion": "2018-09-01",
            "name": "[concat(parameters('privateDnsZones_privatelink_azurewebsites_net_name'), '/theprivateweb')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZones_privatelink_azurewebsites_net_name'))]"
            ],
            "properties": {
                "metadata": {
                    "creator": "created by private endpoint webappprivateendpoint with resource guid 2dc7a952-9088-4ac2-9bfb-d80dbeaab201"
                },
                "ttl": 10,
                "aRecords": [
                    {
                        "ipv4Address": "10.8.5.4"
                    }
                ]
            }
        }

```

##### Private DNS Zone (SOA)

```json

 {
            "type": "Microsoft.Network/privateDnsZones/SOA",
            "apiVersion": "2018-09-01",
            "name": "[concat(parameters('privateDnsZones_privatelink_azurewebsites_net_name'), '/@')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZones_privatelink_azurewebsites_net_name'))]"
            ],
            "properties": {
                "ttl": 3600,
                "soaRecord": {
                    "email": "azureprivatedns-host.microsoft.com",
                    "expireTime": 2419200,
                    "host": "azureprivatedns.net",
                    "minimumTtl": 10,
                    "refreshTime": 3600,
                    "retryTime": 300,
                    "serialNumber": 1
                }
            }
}

```

##### Network Interface

```json

 {
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2020-11-01",
            "name": "[parameters('networkInterfaces_webappprivateendpoint_nic_4abdda20_7c1c_4d99_be0c_5f739a74f6ae_name')]",
            "location": "westeurope",
            "dependsOn": [
                "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_vnet_thenet_name'), 'endpoints')]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "privateEndpointIpConfig.722a3b8d-6af1-4668-bcd7-abb047c943df",
                        "properties": {
                            "privateIPAddress": "10.8.5.4",
                            "privateIPAllocationMethod": "Dynamic",
                            "subnet": {
                                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_vnet_thenet_name'), 'endpoints')]"
                            },
                            "primary": true,
                            "privateIPAddressVersion": "IPv4"
                        }
                    }
                ],
                "dnsSettings": {
                    "dnsServers": []
                },
                "enableAcceleratedNetworking": false,
                "enableIPForwarding": false
            }
        }

```

##### Private Endpoint

```json

 {
            "type": "Microsoft.Network/privateEndpoints",
            "apiVersion": "2020-11-01",
            "name": "[parameters('privateEndpoints_webappprivateendpoint_name')]",
            "location": "westeurope",
            "dependsOn": [
                "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_vnet_thenet_name'), 'endpoints')]"
            ],
            "properties": {
                "privateLinkServiceConnections": [
                    {
                        "name": "[concat(parameters('privateEndpoints_webappprivateendpoint_name'), '-892a')]",
                        "properties": {
                            "privateLinkServiceId": "[parameters('sites_theprivateweb_externalid')]",
                            "groupIds": [
                                "sites"
                            ],
                            "privateLinkServiceConnectionState": {
                                "status": "Approved",
                                "actionsRequired": "None"
                            }
                        }
                    }
                ],
                "manualPrivateLinkServiceConnections": [],
                "subnet": {
                    "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_vnet_thenet_name'), 'endpoints')]"
                },
                "customDnsConfigs": []
            }
        }

```

[Back to top](#table-of-content)