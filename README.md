# HYPERLEDGER FABRIC BLOCKCHAIN NETWORK 
## BLOCKCHAIN AUTOMATION FRAMEWORK 
### CLOUD PROVIDER MICROSOFT AZURE

#### SETTING UP kubectl COMMAND LINE TOOL

```
sudo apt install curl

curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

mkdir -p ~/.kube

cd ~/.kube

```

#### SETTING UP GIT ON YOUR MACHINE

To use Git, you need to install the software on your local machine.
Download and install git bash from http://git-scm.com/downloads.

```
sudo apt-get install git

sudo add-apt-repository ppa:git-core/ppa

sudo apt update

sudo apt install git
```
After the install has completed you can test whether Git has installed correctly by running the command

```
git --version
```

If this works successfully you will need to configure your Git instance by    specifying your username and email address. This is done with the following two commands (Use your GitHub username and email address, if you already have a Github Account):

```
git config --global user.name "<username>"
git config --global user.email "<useremail>"
```

To verify that the username and password was entered correctly, check by running

```
git config user.name
git config user.email
```

#### SETTING UP GITHUB

GitHub is a web-based Git repository hosting service. It offers all of the distributed revision control and source code management (SCM) functionality of Git as well as adding its own features. You can create projects and repositories for you and your teams’ needs.

Complete the following steps to download and configure the BAF repository on your local machine.
- If you already have an account from previously, you can use the same account. If you don’t have an account, create one.
- Go to [blockchain-automation-framework](https://github.com/hyperledger-labs/blockchain-automation-framework) on GitHub and click the Fork button on top right. This will create a copy of the repo to your own GitHub account.In git bash, write and execute the command:

```
ssh-keygen -q -N "" -f ~/.ssh/gitops
```

- This generates an SSH key-pair in your user/.ssh directory: gitops (private key) and gitops.pub (public key).

- Add the public key contents from gitops.pub (starts with ssh-rsa) as an Access Key (with read-write permissions) in your Github repository by following [this guide](https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account) or the below steps.
  - Copy the SSH public key to your clipboard.
  - In the upper-right corner of any page, click your profile photo, then click Settings. 
  - In the user settings sidebar, click SSH and GPG keys. 
  - Click New SSH key or Add SSH key. 
  - In the "Title" field, add a descriptive label for the new key. For example, if you're using a personal Mac, you might call this key "Personal MacBook Air".
  - Paste your key into the "Key" field.
  - Click Add SSH key. 

- Execute the following commands to add the key to your ssh-agent

```
 eval "$(ssh-agent)"
 ssh-add ~/.ssh/gitops
```

#### CLONE THE NECESSARY REPOS

```
mkdir <directory name>

cd <directory name>

git clone git@github.com:sivaramkannan/blockchain-platform.git

git clone git@github.com:sivaramkannan/baf-intain-fork.git

git clone git@github.com:sivaramkannan/BAF-intain-deployment-configs.git

git clone git@github.com:SoundaryaA3098/Deployment-yamls.git
```

#### SETUP THE CLUSTER AND VM

#####SETTING UP AZURE CLI

```
sudo apt-get update

sudo apt-get install ca-certificates curl apt-transport-https lsb-release gnupg

curl -sL https://packages.microsoft.com/keys/microsoft.asc |
    gpg --dearmor |
    sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null

AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" |
    sudo tee /etc/apt/sources.list.d/azure-cli.list

sudo apt-get update

sudo apt-get install azure-cli

az login
```

LOGIN INTO [AZURE PORTAL] (https://portal.azure.com)

AFTER LOGGING INTO THE ABOVE LINK , IN SEARCH TAB SEARCH “Subscriptions”
COPY THE SUBSCRIPTION ID

IN SEARCH TAB SEARCH “Azure Active Directory”
COPY THE TENANT ID

```
export TF_VAR_subscription_id=<subscription_id>
export TF_VAR_tenant_id=<tenant_id>
```

Create Blob store for remote Statefile:
Go the storage account that would be used for this cluster creation.
Create a new container storage (blob store).
Mark the name of the new container created for subsequent usage.
cd blockchain-platform
code cloudplatform
Example Configuration:
# Resource Group
resource_group = "umb-rg"
location = "West US 2"

# Storage Account Name" 
storage_account_name = "umbstorageaccount"

# Blobstore configuration
blobstore_name = "umb-staging-cni-backup"

# Azure AKS configuration
cluster_name = "umb-staging-cni"
cluster_nodepool_name = "nodepool1"
cluster_vmsize = "Standard_D4a_v4"
cluster_nodepool_size = 3

#Vault VM Configuration
prefix = "umb-vault"
vault_vmsize = "Standard_DS1_v2"

Launch the cluster:
```
terraform init - If you are running terraform for the first time
terraform apply
```

Add access details to kubectl: (don't overwrite the default set in ~/.kube/config)

```
echo "$(terraform output kube_config)" > $HOME/.kube/azurek8s
export KUBECONFIG="$HOME/.kube/azurek8s"
```
Destroy the cluster:

```
terraform destroy
```

Cluster login:

```
az aks get-credentials --resource-group <resource group name> --name <aks cluster name>
az aks get-credentials --resource-group umb-rg --name umb-staging-cni
```

#### HOW TO LOGIN TO THE INSTANCE

```
ssh -i .ssh/id_rsa.pub vaultadmin@<vm-public-ip>
```

AFTER LOGGING INTO THE INSTANCE (VM IN AZURE) DO THE FOLLOWING

https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04 
Follow the link to install docker

```
which docker 
cd /usr/bin
sudo chmod -R 777 docker*
```

DO THE FOLLOWING 

1. Create a file /etc/systemd/system/vault-docker.service
   
```
cd  /etc/systemd/system
sudo nano vault-docker.service
```
2. Copy paste the below content there
[Unit]

Description=Vault Docker Application Service
Requires=docker.service
After=docker.service

[Service]

ExecStart=/usr/bin/docker run --rm -v /opt/vault/store:/vault/file -p
8200:8200 --cap-add=IPC_LOCK -e 'VAULT_LOCAL_CONFIG={"backend": {"file":
{"path": "/vault/file"}}, "default_lease_ttl": "168h", "max_lease_ttl":
"720h", "listener": {"tcp": {"address": "0.0.0.0:8200", "tls_disable":
true}}}' vault server
Restart=always
RestartSec=90
StartLimitInterval=400
StartLimitBurst=3

[Install]

WantedBy=multi-user.target

3. sudo systemctl enable vault-docker.service
4. sudo systemctl start vault-docker.service

```dig +short myip.opendns.com @resolver1.opendns.com```
The above command gives a IP . Note it down.

Execute the below commands in your terminal.
```
export VAULT_ADDR='http://DNS_Name:8200' //DROPLET IP
vault operator init -key-shares=1 -key-threshold=1
vault login root-token
export VAULT_TOKEN="<Your Vault root token>"
vault operator unseal
vault secrets enable -version=1 -path=secret kv
vault status
vault secrets list
```




