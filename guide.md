# Create RG
* az group create -n bhiac02 -l westeurope
* West Europe
* Pin to dashboard for easy access

# Create pre-configured Jenkins VM
* https://github.com/Azure/azure-quickstart-templates/tree/master/azure-jenkins
* Click "Deploy to Azure"

# Set up SSH Port Forwarding
* Windows: putty.exe -ssh -L 8080:localhost:8080 <User name>@<Public DNS name of instance you just created>
* Mac/Linux: ssh -L 8080:localhost:8080 <User name>@<Public DNS name of instance you just created>

# SSH to Jenkins VM
* Run /opt/azure_jenkins_config/config_azure.sh and pick option 1
* Login
* Select subscriptons, storage account
* Note down access credentials
* Run sudo cat /var/lib/jenkins/secrets/initialAdminPassword to get password

# Browse to localhost:8080
* Enter admin password from above
* Install recommended plugins
* Create admin user

# Configure Azure VM Agents Plugin
* Manage Jenkins -> Configure System -> Cloud
* Add new credentials
* Kind = Microsoft Azure VM Agents
* Change VM size to Standard_D1_v2
* Add new Username/Password admin account for VMs
* Add following to Init Script

```bash
#Install Azure CLI
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
     sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893
sudo apt-get install apt-transport-https -y
sudo apt-get update && sudo apt-get install azure-cli -y
```

# Set up git locally
* Install Git for Windows/Linux/Mac
* Fork this repository on GitHub

Test login using Service Principal
* az login --service-principal -u <Client-ID> -p <Client-secret> --tenant <Tenant-ID>