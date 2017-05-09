Add the following variables:

"modulesUrl": "https://raw.githubusercontent.com/bhummerstone/iac-hackathon/master/dsc/WebsiteInstall.ps1.zip",
"configurationFunction": "WebsiteInstall.ps1\\Main"


Add the following resource after the Virtual Machine resource:

{
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "DSC",
      "apiVersion": "2015-05-01-preview",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Microsoft.Powershell",
        "type": "DSC",
        "typeHandlerVersion": "2.20",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "ModulesUrl": "[variables('modulesUrl')]",
          "ConfigurationFunction": "[variables('configurationFunction')]",
          "Properties": {
            "MachineName": "[variables('vmName')]"
          }
        },
        "protectedSettings": null
      }
}