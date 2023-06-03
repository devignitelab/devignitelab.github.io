---
title: Virtual Machine Availability Set
tags: 
  - "autoscaling"
  - "virtual-machine"
---

- ARM Template
```json
{
   "$schema":"https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
   "contentVersion":"1.0.0.0",
   "parameters":{
      "lab-name":{
         "type":"string",
         "defaultValue":"104"
      },
      "vm-count":{
         "type":"int",
         "defaultValue":2
      },
      "aset-name":{
         "type":"string",
         "defaultValue":"aset"
      }
   },
   "variables":{
      "lab-id":"[concat('lab-', parameters('lab-name'),'-')]",
      "vmName":"[concat(variables('lab-id'),'aset-vm-')]",
      "nsg-name":"[concat(variables('vmName'),'nsg')]",
      "vnet-name":"[concat(variables('lab-id'),'vnet')]",
      "nic-name":"[concat(variables('vmName'),'nic-')]",
      "availset":"[concat(variables('lab-id'), parameters('aset-name'))]",
      "tag-value":"[concat('lab-', parameters('lab-name'))]"
   },
   "resources":[
      {
         "name":"[variables('availset')]",
         "type":"Microsoft.Compute/availabilitySets",
         "apiVersion":"2019-07-01",
         "location":"[resourceGroup().location]",
         "tags":{
            "lab":"[variables('tag-value')]"
         },
         "properties":{
            "platformFaultDomainCount":2,
            "platformUpdateDomainCount":3
         },
         "sku":{
            "name":"Aligned"
         }
      },
      {
         "name":"[variables('nsg-name')]",
         "type":"Microsoft.Network/networkSecurityGroups",
         "apiVersion":"2018-08-01",
         "location":"[resourceGroup().location]",
         "properties":{
            "securityRules":[
               {
                  "name":"rule-remote",
                  "properties":{
                     "description":"description",
                     "protocol":"Tcp",
                     "sourcePortRange":"*",
                     "destinationPortRange":"22",
                     "sourceAddressPrefix":"*",
                     "destinationAddressPrefix":"*",
                     "access":"Allow",
                     "priority":100,
                     "direction":"Inbound"
                  }
               },
               {
                  "name":"rule-http",
                  "properties":{
                     "description":"description",
                     "protocol":"Tcp",
                     "sourcePortRange":"*",
                     "destinationPortRange":"80",
                     "sourceAddressPrefix":"*",
                     "destinationAddressPrefix":"*",
                     "access":"Allow",
                     "priority":101,
                     "direction":"Inbound"
                  }
               }
            ]
         }
      },
      {
         "name":"[variables('vnet-name')]",
         "type":"Microsoft.Network/virtualNetworks",
         "apiVersion":"2019-11-01",
         "location":"[resourceGroup().location]",
         "dependsOn":[
            "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsg-name'))]"
         ],
         "tags":{
            "lab":"[variables('tag-value')]"
         },
         "properties":{
            "addressSpace":{
               "addressPrefixes":[
                  "10.0.0.0/16"
               ]
            },
            "copy":[
               {
                  "name":"subnets",
                  "count":"[parameters('vm-count')]",
                  "input":{
                     "name":"[concat('subnet-',copyIndex('subnets', 1))]",
                     "properties":{
                        "addressPrefix":"[concat('10.0.',copyIndex('subnets', 1),'.0/24')]",
                        "networkSecurityGroup":{
                           "id":"[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsg-name'))]"
                        }
                     }
                  }
               }
            ]
         }
      },
      {
         "name":"[concat(variables('nic-name'), copyIndex(1))]",
         "type":"Microsoft.Network/networkInterfaces",
         "apiVersion":"2019-11-01",
         "location":"[resourceGroup().location]",
         "dependsOn":[
            "[resourceId('Microsoft.Network/virtualNetworks', variables('vnet-name'))]"
         ],
         "tags":{
            "lab":"[variables('tag-value')]"
         },
         "copy":{
            "name":"nicCopy",
            "count":"[parameters('vm-count')]"
         },
         "properties":{
            "ipConfigurations":[
               {
                  "name":"ip-config",
                  "properties":{
                     "privateIPAllocationMethod":"Dynamic",
                     "subnet":{
                        "id":"[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnet-name'), concat('subnet-', copyIndex(1)))]"
                     }
                  }
               }
            ]
         }
      },
      {
         "name":"[concat(variables('vmName'), copyIndex(1))]",
         "type":"Microsoft.Compute/virtualMachines",
         "apiVersion":"2019-07-01",
         "location":"[resourceGroup().location]",
         "dependsOn":[
            "[resourceId('Microsoft.Network/networkInterfaces', concat(variables('nic-name'), copyIndex(1)))]"
         ],
         "tags":{
            "lab":"[variables('tag-value')]"
         },
         "copy":{
            "name":"vm-copy",
            "count":"[parameters('vm-count')]",
            "mode":"Parallel"
         },
         "properties":{
            "hardwareProfile":{
               "vmSize":"Standard_B1s"
            },
            "osProfile":{
               "computerName":"vmlinux",
               "adminUsername":"nikx",
               "adminPassword":"Password@123"
            },
            "storageProfile":{
               "imageReference":{
                  "publisher":"Canonical",
                  "offer":"UbuntuServer",
                  "sku":"19.10-DAILY",
                  "version":"latest"
               },
               "osDisk":{
                  "name":"[concat('vmdisk-',variables('vmName'), copyIndex(1))]",
                  "caching":"ReadWrite",
                  "createOption":"FromImage"
               }
            },
            "availabilitySet":{
               "id":"[resourceId('Microsoft.Compute/availabilitySets',variables('availset'))]"
            },
            "networkProfile":{
               "networkInterfaces":[
                  {
                     "id":"[resourceId('Microsoft.Network/networkInterfaces', concat(variables('nic-name'), copyIndex(1)))]"
                  }
               ]
            },
            "diagnosticsProfile":{
               "bootDiagnostics":{
                  "enabled":false
               }
            }
         }
      }
   ],
   "outputs":{
      "siteUri":{
         "type":"string",
         "value":"success"
      }
   }
}
```