Add the following variables:

"modulesUrl": "https://raw.githubusercontent.com/<github-username>/iac-hackathon-dsc/master/WebsiteInstall.zip",
"configurationFunction": "Main"


Add the following resource after the Virtual Machine resource:

{
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat(variables('vmName'),'/Microsoft.Powershell.DSC')]",
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
          "configuration": {
            "url": "[variables('modulesUrl')]",
            "script": "WebSiteInstall.ps1",
            "function": "[variables('configurationFunction')]"
          },
          "configurationArguments": {
            "nodeName": "[variables('vmName')]"
          }
        },
        "protectedSettings": null
      }
    }