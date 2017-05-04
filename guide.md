# Infrastructure as Code Hackathon
## Phase 1: Jenkins Deploying to Azure
### Create RG
* az group create -n bhiac01 -l westeurope
* West Europe
* Pin to dashboard for easy access

### Create Jenkins VM
* Azure Portal -> New -> Jenkins (Bitnami)
* A2_v2 is a good size
* New vNet -> 192.168.0.0/23 for address space, 192.168.0.0/24 for subnet
* Leave boot diagnostics enabled

### Initial Jenkins Config
* Browse to public IP of Jenkins server
* Username is user
* Password is shown in boot diagnostics of VM
* Install recommended plugins, restart

### Prepare Jenkins VM
* Install Azure CLI using following code:

```bash
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
     sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893
sudo apt-get update && sudo apt-get install azure-cli -y
```

* Create Azure Service Principal with required permissions:

```bash
az ad sp create-for-rbac --scopes /subscriptions/<subscription-id>/resourceGroups/bhiac01 -n "bhjenkinssp"
```
* Copy AppID, password & tenant for later
* Test login using Service Principal:

```bash
az login --service-principal -u <Client-ID> -p <Client-secret> --tenant <Tenant-ID>
```

### Create Initial Build Pipeline
* Jenkins web browser, new Item
* Name, Freestyle Project -> OK
* General -> GitHub Project -> https://github.com/bhummerstone/iac-hackathon.git/
* Source Code Management -> Git -> https://github.com/bhummerstone/iac-hackathon.git, no authentication required
* Build Triggers -> GitHub hook trigger for GITScm polling
* Build Environment -> Delete workspace before build starts
* Build -> New Build Step -> Execute shell

```bash
az login --service-principal -u <appID> -p <password> --tenant <tenant>
az group deployment create -g bhiac01 --template-file azuredeploy.json --parameters @azuredeploy.parameters.json
```


## Phase 2: 
### Set up git locally
* Install Git for Windows/Linux/Mac
* Fork this repository on GitHub
* Clone it locally from your fork

### Link Jenkins/GitHub
* Got to GitHub -> Profile pic drop down -> Settings -> Personal access tokens
* Generate new token
* admin:repo_hook, repo:status
* Copy key
* Jenkins -> Manage Jenkins -> Configure System -> GitHub Servers
* Add new credential -> Kind: Secret text, paste in personal access token, fill in ID -> Add
* Test connection

### Build ARM template

Test login using Service Principal
* az login --service-principal -u <Client-ID> -p <Client-secret> --tenant <Tenant-ID>