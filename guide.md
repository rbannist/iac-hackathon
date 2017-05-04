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
* A2_v2 is a good size
* New vNet -> 192.168.0.0/23 for address space, 192.168.0.0/24 for subnet
* Leave boot diagnostics enabled

### Initial Jenkins Config
* Browse to public IP of Jenkins server
* Username is user
* Password is shown in boot diagnostics of VM
* Install recommended plugins, restart

### Prepare Jenkins VM
* SSH into Jenkins VM
* Install Azure CLI using following code:

```bash
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
     sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893
sudo apt-get update && sudo apt-get install azure-cli -y
```

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
az group deployment create -g bhiacdeploy01 --template-file templates/azuredeploy.json --parameters \"{\"adminPassword\":{\"value\":\"<secure_password>\"},\"dnsLabelPrefix\":{\"value\":\"<unique_dns>\"}}\"
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


## Phase 3: Desired State Configuration
### Adding DSC to Existing Template
* Edit templates/azuredeploy.json
*  


## Phase 4: Moving to Azure Automation DSC


## Phase 5: More Complex Templates