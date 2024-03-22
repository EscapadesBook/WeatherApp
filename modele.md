{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "staticSites_WeatherApp_name": {
            "defaultValue": "WeatherApp",
            "type": "String"
        },
        "virtualMachines_WeatherAppVM2024_name": {
            "defaultValue": "WeatherAppVM2024",
            "type": "String"
        },
        "firewallPolicies_FireWallWeather_name": {
            "defaultValue": "FireWallWeather",
            "type": "String"
        },
        "networkInterfaces_weatherappvm2024687_name": {
            "defaultValue": "weatherappvm2024687",
            "type": "String"
        },
        "publicIPAddresses_WeatherAppVM2024_ip_name": {
            "defaultValue": "WeatherAppVM2024-ip",
            "type": "String"
        },
        "virtualNetworks_WeatherAppVM2024_vnet_name": {
            "defaultValue": "WeatherAppVM2024-vnet",
            "type": "String"
        },
        "networkSecurityGroups_WeatherAppVM2024_nsg_name": {
            "defaultValue": "WeatherAppVM2024-nsg",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/firewallPolicies",
            "apiVersion": "2023-09-01",
            "name": "[parameters('firewallPolicies_FireWallWeather_name')]",
            "location": "eastus",
            "properties": {
                "sku": {
                    "tier": "Basic"
                },
                "threatIntelMode": "Off",
                "threatIntelWhitelist": {
                    "fqdns": [],
                    "ipAddresses": []
                }
            }
        },
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2023-09-01",
            "name": "[parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name')]",
            "location": "eastus",
            "properties": {
                "securityRules": [
                    {
                        "name": "RDP",
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups/securityRules', parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name'), 'RDP')]",
                        "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                        "properties": {
                            "protocol": "TCP",
                            "sourcePortRange": "*",
                            "destinationPortRange": "3389",
                            "sourceAddressPrefix": "*",
                            "destinationAddressPrefix": "*",
                            "access": "Allow",
                            "priority": 300,
                            "direction": "Inbound",
                            "sourcePortRanges": [],
                            "destinationPortRanges": [],
                            "sourceAddressPrefixes": [],
                            "destinationAddressPrefixes": []
                        }
                    },
                    {
                        "name": "HTTP",
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups/securityRules', parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name'), 'HTTP')]",
                        "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                        "properties": {
                            "protocol": "TCP",
                            "sourcePortRange": "*",
                            "destinationPortRange": "80",
                            "sourceAddressPrefix": "*",
                            "destinationAddressPrefix": "*",
                            "access": "Allow",
                            "priority": 320,
                            "direction": "Inbound",
                            "sourcePortRanges": [],
                            "destinationPortRanges": [],
                            "sourceAddressPrefixes": [],
                            "destinationAddressPrefixes": []
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2023-09-01",
            "name": "[parameters('publicIPAddresses_WeatherAppVM2024_ip_name')]",
            "location": "eastus",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
            "properties": {
                "ipAddress": "52.226.37.83",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Static",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2023-09-01",
            "name": "[parameters('virtualNetworks_WeatherAppVM2024_vnet_name')]",
            "location": "eastus",
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "10.1.0.0/16"
                    ]
                },
                "subnets": [
                    {
                        "name": "default",
                        "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_WeatherAppVM2024_vnet_name'), 'default')]",
                        "properties": {
                            "addressPrefix": "10.1.0.0/24",
                            "delegations": [],
                            "privateEndpointNetworkPolicies": "Disabled",
                            "privateLinkServiceNetworkPolicies": "Enabled"
                        },
                        "type": "Microsoft.Network/virtualNetworks/subnets"
                    }
                ],
                "virtualNetworkPeerings": [],
                "enableDdosProtection": false
            }
        },
        {
            "type": "Microsoft.Web/staticSites",
            "apiVersion": "2023-01-01",
            "name": "[parameters('staticSites_WeatherApp_name')]",
            "location": "East US 2",
            "sku": {
                "name": "Free",
                "tier": "Free"
            },
            "properties": {
                "repositoryUrl": "[concat('https://github.com/EscapadesBook/', parameters('staticSites_WeatherApp_name'))]",
                "branch": "main",
                "stagingEnvironmentPolicy": "Enabled",
                "allowConfigFileUpdates": true,
                "provider": "GitHub",
                "enterpriseGradeCdnStatus": "Disabled"
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2023-03-01",
            "name": "[parameters('virtualMachines_WeatherAppVM2024_name')]",
            "location": "eastus",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkInterfaces_weatherappvm2024687_name'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "Standard_B1s"
                },
                "additionalCapabilities": {
                    "hibernationEnabled": false
                },
                "storageProfile": {
                    "imageReference": {
                        "publisher": "MicrosoftWindowsServer",
                        "offer": "WindowsServer",
                        "sku": "2019-datacenter-gensecond",
                        "version": "latest"
                    },
                    "osDisk": {
                        "osType": "Windows",
                        "name": "[concat(parameters('virtualMachines_WeatherAppVM2024_name'), '_disk1_ee9f4cc765a5475782472e1a01df3eeb')]",
                        "createOption": "FromImage",
                        "caching": "ReadWrite",
                        "managedDisk": {
                            "storageAccountType": "Premium_LRS",
                            "id": "[resourceId('Microsoft.Compute/disks', concat(parameters('virtualMachines_WeatherAppVM2024_name'), '_disk1_ee9f4cc765a5475782472e1a01df3eeb'))]"
                        },
                        "deleteOption": "Delete",
                        "diskSizeGB": 127
                    },
                    "dataDisks": [],
                    "diskControllerType": "SCSI"
                },
                "osProfile": {
                    "computerName": "WeatherAppVM202",
                    "adminUsername": "azureuser",
                    "windowsConfiguration": {
                        "provisionVMAgent": true,
                        "enableAutomaticUpdates": true,
                        "patchSettings": {
                            "patchMode": "AutomaticByOS",
                            "assessmentMode": "ImageDefault",
                            "enableHotpatching": false
                        },
                        "enableVMAgentPlatformUpdates": false
                    },
                    "secrets": [],
                    "allowExtensionOperations": true,
                    "requireGuestProvisionSignal": true
                },
                "securityProfile": {
                    "uefiSettings": {
                        "secureBootEnabled": true,
                        "vTpmEnabled": true
                    },
                    "securityType": "TrustedLaunch"
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkInterfaces_weatherappvm2024687_name'))]",
                            "properties": {
                                "deleteOption": "Detach"
                            }
                        }
                    ]
                }
            }
        },
        {
            "type": "Microsoft.Network/networkSecurityGroups/securityRules",
            "apiVersion": "2023-09-01",
            "name": "[concat(parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name'), '/HTTP')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name'))]"
            ],
            "properties": {
                "protocol": "TCP",
                "sourcePortRange": "*",
                "destinationPortRange": "80",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 320,
                "direction": "Inbound",
                "sourcePortRanges": [],
                "destinationPortRanges": [],
                "sourceAddressPrefixes": [],
                "destinationAddressPrefixes": []
            }
        },
        {
            "type": "Microsoft.Network/networkSecurityGroups/securityRules",
            "apiVersion": "2023-09-01",
            "name": "[concat(parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name'), '/RDP')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name'))]"
            ],
            "properties": {
                "protocol": "TCP",
                "sourcePortRange": "*",
                "destinationPortRange": "3389",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 300,
                "direction": "Inbound",
                "sourcePortRanges": [],
                "destinationPortRanges": [],
                "sourceAddressPrefixes": [],
                "destinationAddressPrefixes": []
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2023-09-01",
            "name": "[concat(parameters('virtualNetworks_WeatherAppVM2024_vnet_name'), '/default')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_WeatherAppVM2024_vnet_name'))]"
            ],
            "properties": {
                "addressPrefix": "10.1.0.0/24",
                "delegations": [],
                "privateEndpointNetworkPolicies": "Disabled",
                "privateLinkServiceNetworkPolicies": "Enabled"
            }
        },
        {
            "type": "Microsoft.Web/staticSites/basicAuth",
            "apiVersion": "2023-01-01",
            "name": "[concat(parameters('staticSites_WeatherApp_name'), '/default')]",
            "location": "East US 2",
            "dependsOn": [
                "[resourceId('Microsoft.Web/staticSites', parameters('staticSites_WeatherApp_name'))]"
            ],
            "properties": {
                "applicableEnvironmentsMode": "SpecifiedEnvironments"
            }
        },
        {
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2023-09-01",
            "name": "[parameters('networkInterfaces_weatherappvm2024687_name')]",
            "location": "eastus",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_WeatherAppVM2024_ip_name'))]",
                "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_WeatherAppVM2024_vnet_name'), 'default')]",
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name'))]"
            ],
            "kind": "Regular",
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "id": "[concat(resourceId('Microsoft.Network/networkInterfaces', parameters('networkInterfaces_weatherappvm2024687_name')), '/ipConfigurations/ipconfig1')]",
                        "etag": "W/\"a03461cd-54a8-4b76-b4b5-537ec7c1e3bb\"",
                        "type": "Microsoft.Network/networkInterfaces/ipConfigurations",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "privateIPAddress": "10.1.0.4",
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_WeatherAppVM2024_ip_name'))]",
                                "properties": {
                                    "deleteOption": "Detach"
                                }
                            },
                            "subnet": {
                                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_WeatherAppVM2024_vnet_name'), 'default')]"
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
                "enableIPForwarding": false,
                "disableTcpStateTracking": false,
                "networkSecurityGroup": {
                    "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroups_WeatherAppVM2024_nsg_name'))]"
                },
                "nicType": "Standard",
                "auxiliaryMode": "None",
                "auxiliarySku": "None"
            }
        }
    ]
