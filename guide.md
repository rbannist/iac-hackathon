# Infrastructure as Code Hackathon
## Phase 1: Building the Pipeline
### Create Resource Groups
* One RG for Jenkins, one for deployed resources
```bash
az group create -n bhjenkins01 -l westeurope
az group create -n bhiacdeploy01 -l westeurope
```

### Create Jenkins VM
* Azure Portal -> New -> Jenkins (Bitnami)
* Deploy to Jenkins RG
* Managed Disks, always!
* A2_v2 is a good size
* New vNet -> 192.168.0.0/23 for address space, 192.168.0.0/24 for subnet
* Leave boot diagnostics enabled

### Initial Jenkins Config
* Browse to public IP of Jenkins server (can see in  properties of VM)
* Username is "user"
* Password is shown in boot diagnostics of VM
* Install recommended plugins, restart

### Prepare Jenkins VM
* SSH into Jenkins VM using public IP
* Install Azure CLI using following code:

```bash
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
     sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893
sudo apt-get update && sudo apt-get install azure-cli -y
```

* Login to Azure CLI using "az login"
* Create Azure Service Principal with required permissions:
```bash
az ad sp create-for-rbac --scopes /subscriptions/<subscription-id>/resourceGroups/bhiacdeploy01 -n "bhjenkinssp"
```

* Copy AppID, password & tenant for later
* Test login using Service Principal:
```bash
az login --service-principal -u <appID> -p <password> --tenant <tenant>
```

### Create Initial Build Pipeline
* Jenkins web browser, New Item
* Name, Freestyle Project -> OK
* General -> GitHub Project -> https://github.com/bhummerstone/iac-hackathon.git
* Source Code Management -> Git -> https://github.com/bhummerstone/iac-hackathon.git, no authentication required
* Build Triggers -> GitHub hook trigger for GITScm polling
* Build Environment -> Delete workspace before build starts
* Build -> New Build Step -> Execute shell

```bash
az login --service-principal -u <appID> -p <password> --tenant <tenant>
az group deployment create -g <resource-group> --template-file templates/azuredeploy.json --parameters '{"adminPassword":{"value":"<secure_password>"},"dnsLabelPrefix":{"value":"<unique_dns>"}}' --verbose
```

* Save, Build Now
* Click on Build Number, check out Console Output
* Also check out deployment process in Azure Portal


## Phase 2: Configuring Continuous Deployment
### Setup Git Locally
* Install Git for Windows/Linux/Mac
* Fork this repository on GitHub
* Clone it locally from your fork

### Link Jenkins/GitHub
* Go to GitHub -> Profile pic drop down -> Settings -> Personal access tokens
* Generate new token
* admin:repo_hook, repo:status
* Copy key
* Jenkins -> Manage Jenkins -> Configure System -> GitHub Servers
* Add new credential -> Kind: Secret text, paste in personal access token, fill in ID -> Add
* Test connection
* Advanced -> Re-register hooks for all jobs

### Test Build Process
* Edit templates/azuredeploy.json locally
* Change VM size from Standard\_A2 to Standard\_A3
* Commit change, push to repository
* Confirm build kicks off in Jenkins and VM size changes in Azure Portal


## Phase 3: Integrating Desired State Configuration
### Getting Started
* Fork https://github.com/bhummerstone/iac-hackathon-dsc repo
* Clone to local machine

### Adding DSC to Existing Template
* In iac-hackathon repo, copy code from dsc-in-template.md into templates/azuredeploy.json
* Replace <github-username> with your GitHub username
* Commit and push the changes
* Browse to public IP of VM
* RDP into the VM and have a look in event logs to see DSC events: Applications and Services Logs/Microsoft/Windows/Desired State Configuration

### Updating the DSC Configuration
* Current example website is a bit boring (sorry Rick) so let's change it up
* Append "-iac" to modulesURL and dscScript variables; this references a different DSC module
* Commit and push
* Browse back to public IP of VM to see changes


## Phase 4: Azure Automation DSC
### Setting up Azure Automation DSC
* In the Portal, New -> Automation -> Create
* Name, put it in the Jenkins RG
* Browse to Automation Account -> DSC Configurations
* Add WebsiteInstall-aadsc.ps1 from iac-hackathon-dsc/originals
* Once published, select Configuration -> Compile. This generates the configuration MOF and uploads it to the Pull server

### Registering Nodes to Azure Automation
* Create new VM -> WS 2016 Datacentre
* Go to Automation Account -> DSC Nodes -> Add Azure VM
* Add VM
* Node Configuration = WebServer.WebServer
* Tick boxes for Reboot and Module Overwrite
* Browse to VM's public IP, confirm that changes have taken effect