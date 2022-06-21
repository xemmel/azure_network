## Azure networking

## Table of Content

4. [Private Endpoints](#private-endpoints)
7. [WebApp Network tests](#webapp-network-tests)
8. [VM scripts](#vm-scripts)
9. [General scripts](#general-scripts)

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


### WebApp Network tests

```powershell
### Test outbound Internet
tcpping www.jp.dk 

### nslookup
nameresolver theprivateweb.azurewebsites.net
nameresolver intdatabase.database.windows.net

```


[Back to top](#table-of-content)

### VM scripts

#### Get Token

```powershell

Clear-Host;

$baseUrl = "http://169.254.169.254/metadata/identity/oauth2/token";
$parameters = "?api-version=2018-02-01&resource=https://graph.microsoft.com/";
## $parameters = "?api-version=2018-02-01&resource=https://management.azure.com/";
##$parameters = "?api-version=2018-02-01&resource=https://storage.azure.com/";
$parameters = "?api-version=2018-02-01&resource=https://database.windows.net/";


$url = "$($baseUrl)$($parameters)";

$headers = @{"metadata" = "true"};

$response = invoke-webrequest -Uri $url -Headers $headers;
## $response.Content;
$token = $null;
$token = $response.Content | ConvertFrom-Json | Select-Object -ExpandProperty access_token;
$token;


```
[Back to top](#table-of-content)

#### Invoke Sql with token

```powershell

Clear-Host;
# Install-Module -Name SqlServer -Force;
Invoke-Sqlcmd `
    -ServerInstance "intdatabase.database.windows.net" `
    -AccessToken $token `
    -Database "users" `
    -Query "select name from sys.database_principals;";

Clear-Host;
    $spName = "win11";
    Invoke-Sqlcmd `
    -ServerInstance "intdatabase.database.windows.net" `
    -AccessToken $token `
    -Database "users" `
    -Query "CREATE USER [$($spName)] FROM EXTERNAL PROVIDER;";

Clear-Host;
    $spName = "win11";
    Invoke-Sqlcmd `
    -ServerInstance "intdatabase.database.windows.net" `
    -AccessToken $token `
    -Database "users" `
    -Query "DROP USER [$($spName)];";

```

[Back to top](#table-of-content)

#### Storage Account commands with Token

##### List Containers

```powershell

Clear-Host;
$storageAccountName = "intprivatestorage";
$saUrl = "https://$($storageAccountName).blob.core.windows.net";
$command = "/?comp=list";
$url = "$($saUrl)$($command)";


$headers = @{"Authorization" = "Bearer $token"; "x-ms-version" = "2021-06-08"};

$response = $null;
$response = invoke-webrequest -Uri $url -Headers $headers;
$response.Content

```

##### List Blobs

```powershell

Clear-Host;
$storageAccountName = "intprivatestorage";
$containerName = "public";
$saUrl = "https://$($storageAccountName).blob.core.windows.net/$($containerName)?restype=container`&comp=list";
#$command = "/?comp=list";
$url = "$($saUrl)";
$date = [System.DateTime]::Now.ToUniversalTime().ToString("r");

$headers = @{
    "Authorization" = "Bearer $token"; 
    "x-ms-version" = "2021-06-08"; 
    "x-ms-date" = $date;
    };

$response = $null;
$response = invoke-webrequest -Uri $url -Headers $headers;
$response.Content

```

##### Put Blob

```powershell

Clear-Host;
$storageAccountName = "intprivatestorage";
$containerName = "public";
$blobName = "test.txt";
$body = @"
  Hello

"@
$date = [System.DateTime]::Now.ToUniversalTime().ToString("r");
$url = "https://$($storageAccountName).blob.core.windows.net/$($containerName)/$($blobName)";



$headers = @{
    "Authorization" = "Bearer $token"; 
    "x-ms-version" = "2021-06-08"; 
    "x-ms-date" = $date;
    "x-ms-blob-type" =  "BlockBlob";
    };

$response = $null;
$response = invoke-webrequest -Method Put -Uri $url -Body $body -Headers $headers;
$response.Content

```

[Back to top](#table-of-content)



### General scripts

#### Translate Token

```powershell

$p = $token.Split('.');
$parts = $p[0];
if (($parts.Length % 4) -ne 0) {
    if (($parts.Length % 4) -eq 1) {
        $parts += "===";
    }
        if (($parts.Length % 4) -eq 2) {
        $parts += "==";
    }
            if (($parts.Length % 4) -eq 3) {
        $parts += "=";
    }
}

[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($parts)) | ConvertFrom-Json | ConvertTo-Json;

$p = $token.Split('.');
$parts = $p[1];
if (($parts.Length % 4) -ne 0) {
    if (($parts.Length % 4) -eq 1) {
        $parts += "===";
    }
        if (($parts.Length % 4) -eq 2) {
        $parts += "==";
    }
            if (($parts.Length % 4) -eq 3) {
        $parts += "=";
    }
}

[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($parts)) | ConvertFrom-Json | ConvertTo-Json;

```


[Back to top](#table-of-content)

#### Format Xml

```powershell

$xml = [xml]($response.Content.Substring(3));
$Indent = 2
$StringWriter = New-Object System.IO.StringWriter
$XmlWriter = New-Object System.XMl.XmlTextWriter $StringWriter
$xmlWriter.Formatting = "indented"
$xmlWriter.Indentation = $Indent
$xml.WriteContentTo($XmlWriter)
$XmlWriter.Flush()
$StringWriter.Flush()
$StringWriter.ToString()


```

[Back to top](#table-of-content)
