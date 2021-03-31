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

##### SETTING UP AZURE CLI

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
#### Resource Group
resource_group = "umb-rg"
location = "West US 2"

#### Storage Account Name" 
storage_account_name = "umbstorageaccount"

#### Blobstore configuration
blobstore_name = "umb-staging-cni-backup"

#### Azure AKS configuration
cluster_name = "umb-staging-cni"
cluster_nodepool_name = "nodepool1"
cluster_vmsize = "Standard_D4a_v4"
cluster_nodepool_size = 3

#### Vault VM Configuration
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

UPDATE THE NETWORK CONFIGURATION FILE WITH THE VAULT DETAILS

#### BAF SETUP

```
cd baf-intain-fork

git checkout -b <branch-name> v0.7.0.0
```
- Create a directory with the same name as the newly created branch under the path “platforms/hyperledger-fabric/releases”.

- CREATE A NEW DIRECTORY CALLED     build

```
cd baf-intain-fork

mkdir build

cd build

cd ../platforms/hyperledger-fabric/configuration/samples/
```

- COPY THE network-fabricv2.yaml FILE IN YOUR BUILD DIRECTORY AND RENAME IT AS network.yaml

- INSIDE THE build DIRECTORY , YOU HAVE TO PLACE 4 FILES.
        * network.yaml, * gitops (private key), * gitops.pub (public key), 
config ( Paste the cluster configuration here )

Copy .ssh/gitops, ..sh/gitops.pub, .kube/config files under the build folder. Make sure the gitops.pub key is added to the git repository where the baf is going to push the files during network deployment.



- UPDATE THE CLOUD PROVIDER, k8s , VAULT SECTION, GITOPS

Make the changes in the following 5 files 

1) baf-intain-fork/platforms/hyperledger-fabric/configuration/roles/create/storageclass/templates/azurepeer_sc.tpl (change the storageaccounttype from Premium_LRS to Standard_LRS)
2) baf-intain-fork/platforms/hyperledger-fabric/configuration/roles/create/storageclass/templates/azureorderer_sc.tpl (change the storageaccounttype from Premium_LRS to Standard_LRS)
3) baf-intain-fork/platforms/hyperledger-fabric/configuration/roles/helm_component/templates/orderernode.tpl (change the storagesize to the required configuration)
4) baf-intain-fork/platforms/hyperledger-fabric/configuration/roles/helm_component/templates/value_peer.tpl (change the storagesize to the required configuration)
5) baf-intain-fork/platforms/hyperledger-fabric/configuration/roles/create/configtx/templates/configtxinit.tpl (Make LifecycleEndorsement and Endorsement from MAJORITY to ANY)

 Prepare the k8s cluster

 Change the branch name, vault configuration, cluster-context in the network.yaml

Update the DNS

UPDATE THE network.yaml 

### FOR EXAMPLE

~~---
# This is a sample configuration file for SupplyChain App which has 5 nodes.
network:
  # Network level configuration specifies the attributes required for each organization
  # to join an existing network.
  type: fabric
  version: 2.2.0                 # currently tested 1.4.0 and 1.4.4

  frontend: disabled #Flag for frontend to enabled for nodes/peers
  
  #Environment section for Kubernetes setup
  env:
    type: "dev"              # tag for the environment. Important to run multiple flux on single cluster
    proxy: haproxy                  # values can be 'haproxy' or 'ambassador'
    ambassadorPorts: 15010,15020    # Any additional Ambassador ports can be given here, must be comma-separated without spaces, this is valid only if proxy='ambassador'
    retry_count: 20                 # Retry count for the checks
    external_dns: disabled           # Should be enabled if using external-dns for automatic route configuration

  # Docker registry details where images are stored. This will be used to create k8s secrets
  # Please ensure all required images are built and stored in this registry.
  # Do not check-in docker_password.
  docker:
    url: "index.docker.io/hyperledgerlabs"
    username: "docker_username"
    password: "docker_password"
  
  # Remote connection information for orderer (will be blank or removed for orderer hosting organization)
  # For RAFT consensus, have odd number (2n+1) of orderers for consensus agreement to have a majority.
  orderers:
    - orderer:
      type: orderer
      name: orderer1
      org_name: intain  #org_name should match one organization definition below in organizations: key            
      uri: orderer1.intain-net:7050   # Can be external or internal URI for orderer which should be reachable by all peers
      certificate: /home/blockchain-automation-framework/build/orderer1.crt           # Ensure that the directory exists

  
  # The channels defined for a network with participating peers in each channel
  channels:
  - channel:
    consortium: UMBConsortium
    channel_name: UmbChannel
    orderer: 
      name: intain
    participants:
    - organization:
      name: trustee
      type: creator       # creator organization will create the channel and instantiate chaincode, in addition to joining the channel and install chaincode
      org_status: new
      peers:
      - peer:
        name: peer0
        gossipAddress: peer0.trustee-net:7051 # External or internal URI of the gossip peer
        peerAddress: peer0.trustee-net:7051
      ordererAddress: orderer1.intain-net:7050             # External or internal URI of the orderer
    - organization:      
      name: investor
      type: joiner        # joiner organization will only join the channel and install chaincode
      org_status: new
      peers:
      - peer:
        name: peer0
        gossipAddress: peer0.investor-net:7051
        peerAddress: peer0.investor-net:7051
      ordererAddress: orderer1.intain-net:7050
    - organization:      
      name: servicer
      type: joiner        # joiner organization will only join the channel and install chaincode
      org_status: new
      peers:
      - peer:
        name: peer0
        gossipAddress: peer0.servicer-net:7051
        peerAddress: peer0.servicer-net:7051
      ordererAddress: orderer1.intain-net:7050
    endorsers:
      name:
      - trustee
      - investor
      - servicer
      corepeerAddress:
      - peer0.trustee-net:7051
      - peer0.investor-net:7051
      - peer0.servicer-net:7051
    genesis:
      name: OrdererGenesis

  
  # Allows specification of one or many organizations that will be connecting to a network.
  # If an organization is also hosting the root of the network (e.g. doorman, membership service, etc),
  # then these services should be listed in this section as well.
  organizations:

    # Specification for the 1st organization. Each organization maps to a VPC and a separate k8s cluster
    - organization:
      name: intain
      country: UK
      state: London
      location: London
      subject: "O=Orderer,L=51.50/-0.13/London,C=GB"
      type: orderer
      external_url_suffix: intain-net.umbdev.intainabs.com
      org_status: new
      cli: enabled
      ca_data:
        url: ca.intain-net:7054
        certificate: file/server.crt        # This has not been implemented in 0.2.0.0
  
      cloud_provider: azure   # Options: aws, azure, gcp, minikube
      aws:
        access_key: "aws_access_key"        # AWS Access key, only used when cloud_provider=aws
        secret_key: "aws_secret_key"        # AWS Secret key, only used when cloud_provider=aws
  
      # Kubernetes cluster deployment variables. The config file path and name has to be provided in case
      # the cluster has already been created.
      k8s:
        region: "West US 2"
        context: "umb-staging-cni"
        config_file: "/home/blockchain-automation-framework/build/config"

      # Hashicorp Vault server address and root-token. Vault should be unsealed.
      # Do not check-in root_token
      vault:
        url: "http://umb-vault.westus2.cloudapp.azure.com:8200"
        root_token: "s.GKqdVOujk3sv2tN12UUCUmy1"
      # Git Repo details which will be used by GitOps/Flux.
      # Do not check-in git_access_token
      gitops:
        git_ssh: "git@github.com:sivaramkannan/baf-intain-fork.git"         # Gitops ssh url for flux value files 
        branch: "umb-staging-cni"           # Git branch where release is being made
        release_dir: "platforms/hyperledger-fabric/releases/umb-staging-cni" # Relative Path in the Git repo for flux sync per environment. 
        chart_source: "platforms/hyperledger-fabric/charts"     # Relative Path where the Helm charts are stored in Git repo
        git_push_url: "github.com/sivaramkannan/baf-intain-fork.git"   # Gitops https URL for git push  (without https://)
        username: "sivaramkannan"          # Git Service user who has rights to check-in in all branches
        password: "ea8e98364360d504d7e78f8be06bf2b4c1955eb2"          # Git Server user password
        email: "sivaram.kannan@intainft.com"                # Email to use in git config
        private_key: "/home/blockchain-automation-framework/build/gitops"          # Path to private key file which has write-access to the git repo
     
      # Services maps to the pods that will be deployed on the k8s cluster
      # This sample is an orderer service and includes a zk-kafka consensus
      services:
        ca:
          name: ca
          subject: "/C=GB/ST=London/L=London/O=Orderer/CN=ca.intain-net"
          type: ca
          grpc:
            port: 7054
        
        consensus:
          name: raft
          type: broker        #This field is not consumed for raft consensus
          replicas: 4         #This field is not consumed for raft consensus
          grpc:
            port: 9092        #This field is not consumed for raft consensus
                
        orderers:
        # This sample has multiple orderers as an example.
        # You can use a single orderer for most production implementations.
        # For RAFT consensus, have odd number (2n+1) of orderers for consensus agreement to have a majority.
        - orderer:
          name: orderer1
          type: orderer
          consensus: raft
          grpc:
            port: 7050   
        
    - organization:
      name: trustee
      country: GB
      state: London
      location: London
      subject: "O=Trustee,OU=Trustee,L=51.50/-0.13/London,C=GB"
      type: peer
      external_url_suffix: trustee-net.umbdev.intainabs.com
      org_status: new
      cli: enabled
      ca_data:
        url: ca.trustee-net:7054
        certificate: file/server.crt
      
      cloud_provider: azure   # Options: aws, azure, gcp, minikube
      aws:
        access_key: "aws_access_key"        # AWS Access key, only used when cloud_provider=aws
        secret_key: "aws_secret_key"        # AWS Secret key, only used when cloud_provider=aws
  
      # Kubernetes cluster deployment variables. The config file path and name has to be provided in case
      # the cluster has already been created.
      k8s:
        region: "West US 2"
        context: "umb-staging-cni"
        config_file: "/home/blockchain-automation-framework/build/config"

      # Hashicorp Vault server address and root-token. Vault should be unsealed.
      # Do not check-in root_token
      vault:
        url: "http://umb-vault.westus2.cloudapp.azure.com:8200"
        root_token: "s.GKqdVOujk3sv2tN12UUCUmy1"
      # Git Repo details which will be used by GitOps/Flux.
      # Do not check-in git_access_token
      gitops:
        git_ssh: "git@github.com:sivaramkannan/baf-intain-fork.git"         # Gitops ssh url for flux value files 
        branch: "umb-staging-cni"           # Git branch where release is being made
        release_dir: "platforms/hyperledger-fabric/releases/umb-staging-cni" # Relative Path in the Git repo for flux sync per environment. 
        chart_source: "platforms/hyperledger-fabric/charts"     # Relative Path where the Helm charts are stored in Git repo
        git_push_url: "github.com/sivaramkannan/baf-intain-fork.git"   # Gitops https URL for git push  (without https://)
        username: "sivaramkannan"          # Git Service user who has rights to check-in in all branches
        password: "ea8e98364360d504d7e78f8be06bf2b4c1955eb2"          # Git Server user password
        email: "sivaram.kannan@intainft.com"                # Email to use in git config
        private_key: "/home/blockchain-automation-framework/build/gitops"          # Path to private key file which has write-access to the git repo

      services:
        ca:
          name: ca
          subject: "/C=GB/ST=London/L=London/O=Trustee/CN=ca.trustee-net"
          type: ca
          grpc:
            port: 7054
        peers:
        - peer:
          name: peer0          
          type: anchor    # This can be anchor/nonanchor. Atleast one peer should be anchor peer.    
          gossippeeraddress: peer0.trustee-net:7051 # Internal Address of the other peer in same Org for gossip, same peer if there is only one peer          
          peerAddress: peer0.trustee-net:7051
          cli: enabled
          grpc:
            port: 7051         
          events:
            port: 7053
          couchdb:
            port: 5984
          restserver:
            targetPort: 20001
            port: 20001 
          expressapi:
            targetPort: 3000
            port: 3000
          chaincode:
            name: "fabcar" #This has to be replaced with the name of the chaincode
            version: "1" #This has to be replaced with the version of the chaincode
            maindirectory: "fabric-sample/chaincode"  #The main directory where chaincode is needed to be placed
            lang: "golang"  # The language in which the chaincode is written ( golang/java/node )
            repository:
              username: "sivaramsk"          # Git Service user who has rights to check-in in all branches
              password: "53c9765ad883b0ab50ff7ca16f8b877597581de6"
              url: "git@github.com:sivaramsk/fabric-samples.git"
              branch: develop
              path: "fabcar/go/"   #The path to the chaincode 
            arguments: '{Make: "Toyota", Model: "Prius", Colour: "blue", Owner: "Tomoko"}' #Arguments to be passed along with the chaincode parameters
            endorsements: "" #Endorsements (if any) provided along with the chaincode 

    - organization:
      name: investor
      country: GB
      state: London
      location: London
      subject: "O=Investor,OU=Investor,L=51.50/-0.13/London,C=GB"
      type: peer
      external_url_suffix: investor-net.umbdev.intainabs.com
      org_status: new
      cli: enabled
      ca_data:
        url: ca.investor-net:7054
        certificate: file/server.crt
      
      cloud_provider: azure   # Options: aws, azure, gcp, minikube
      aws:
        access_key: "aws_access_key"        # AWS Access key, only used when cloud_provider=aws
        secret_key: "aws_secret_key"        # AWS Secret key, only used when cloud_provider=aws
  
      # Kubernetes cluster deployment variables. The config file path and name has to be provided in case
      # the cluster has already been created.
      k8s:
        region: "West US 2"
        context: "umb-staging-cni"
        config_file: "/home/blockchain-automation-framework/build/config"

      # Hashicorp Vault server address and root-token. Vault should be unsealed.
      # Do not check-in root_token
      vault:
        url: "http://umb-vault.westus2.cloudapp.azure.com:8200"
        root_token: "s.GKqdVOujk3sv2tN12UUCUmy1"
      # Git Repo details which will be used by GitOps/Flux.
      # Do not check-in git_access_token
      gitops:
        git_ssh: "git@github.com:sivaramkannan/baf-intain-fork.git"         # Gitops ssh url for flux value files 
        branch: "umb-staging-cni"           # Git branch where release is being made
        release_dir: "platforms/hyperledger-fabric/releases/umb-staging-cni" # Relative Path in the Git repo for flux sync per environment. 
        chart_source: "platforms/hyperledger-fabric/charts"     # Relative Path where the Helm charts are stored in Git repo
        git_push_url: "github.com/sivaramkannan/baf-intain-fork.git"   # Gitops https URL for git push  (without https://)
        username: "sivaramkannan"          # Git Service user who has rights to check-in in all branches
        password: "ea8e98364360d504d7e78f8be06bf2b4c1955eb2"          # Git Server user password
        email: "sivaram.kannan@intainft.com"                # Email to use in git config
        private_key: "/home/blockchain-automation-framework/build/gitops"          # Path to private key file which has write-access to the git repo

      services:
        ca:
          name: ca
          subject: "/C=GB/ST=London/L=London/O=Investor/CN=ca.investor-net"
          type: ca
          grpc:
            port: 7054
        peers:
        - peer:
          name: peer0          
          type: anchor    # This can be anchor/nonanchor. Atleast one peer should be anchor peer.    
          gossippeeraddress: peer0.investor-net:7051 # Internal Address of the other peer in same Org for gossip, same peer if there is only one peer          
          peerAddress: peer0.investor-net:7051
          cli: enabled
          grpc:
            port: 7051         
          events:
            port: 7053
          couchdb:
            port: 5984
          restserver:
            targetPort: 20001
            port: 20001 
          expressapi:
            targetPort: 3000
            port: 3000
          chaincode:
            name: "fabcar" #This has to be replaced with the name of the chaincode
            version: "1" #This has to be replaced with the version of the chaincode
            maindirectory: "fabric-sample/chaincode"  #The main directory where chaincode is needed to be placed
            lang: "golang"  # The language in which the chaincode is written ( golang/java/node )
            repository:
              username: "sivaramsk"          # Git Service user who has rights to check-in in all branches
              password: "53c9765ad883b0ab50ff7ca16f8b877597581de6"
              url: "git@github.com:sivaramsk/fabric-samples.git"
              branch: develop
              path: "fabcar/go/"   #The path to the chaincode 
            arguments: '{Make: "Toyota", Model: "Prius", Colour: "blue", Owner: "Tomoko"}' #Arguments to be passed along with the chaincode parameters
            endorsements: "" #Endorsements (if any) provided along with the chaincode 
 
    - organization:
      name: servicer
      country: US
      state: New York
      location: New York
      subject: "O=Servicer,OU=Servicer,L=40.73/-74/New York,C=US"
      type: peer
      external_url_suffix: servicer-net.umbdev.intainabs.com
      org_status: new
      cli: enabled
      ca_data:
        url: ca.servicer-net:7054
        certificate: file/server.crt
      
      cloud_provider: azure   # Options: aws, azure, gcp, minikube
      aws:
        access_key: "aws_access_key"        # AWS Access key, only used when cloud_provider=aws
        secret_key: "aws_secret_key"        # AWS Secret key, only used when cloud_provider=aws
  
      # Kubernetes cluster deployment variables. The config file path and name has to be provided in case
      # the cluster has already been created.
      k8s:
        region: "West US 2"
        context: "umb-staging-cni"
        config_file: "/home/blockchain-automation-framework/build/config"

      # Hashicorp Vault server address and root-token. Vault should be unsealed.
      # Do not check-in root_token
      vault:
        url: "http://umb-vault.westus2.cloudapp.azure.com:8200"
        root_token: "s.GKqdVOujk3sv2tN12UUCUmy1"
      # Git Repo details which will be used by GitOps/Flux.
      # Do not check-in git_access_token
      gitops:
        git_ssh: "git@github.com:sivaramkannan/baf-intain-fork.git"         # Gitops ssh url for flux value files 
        branch: "umb-staging-cni"           # Git branch where release is being made
        release_dir: "platforms/hyperledger-fabric/releases/umb-staging-cni" # Relative Path in the Git repo for flux sync per environment. 
        chart_source: "platforms/hyperledger-fabric/charts"     # Relative Path where the Helm charts are stored in Git repo
        git_push_url: "github.com/sivaramkannan/baf-intain-fork.git"   # Gitops https URL for git push  (without https://)
        username: "sivaramkannan"          # Git Service user who has rights to check-in in all branches
        password: "ea8e98364360d504d7e78f8be06bf2b4c1955eb2"          # Git Server user password
        email: "sivaram.kannan@intainft.com"                # Email to use in git config
        private_key: "/home/blockchain-automation-framework/build/gitops"          # Path to private key file which has write-access to the git repo


      #Optional for infrastructure configuration files.
      infrastructure:
        target_state: "present"  # Options: present, absent, planned            
        refresh_inventory: yes  

        
      services:
        ca:
          name: ca
          subject: "/C=US/ST=New York/L=New York/O=Servicer/CN=ca.servicer-net"
          type: ca
          grpc:
            port: 7054
        peers:
        - peer:
          name: peer0          
          type: anchor    # This can be anchor/nonanchor. Atleast one peer should be anchor peer. 
          gossippeeraddress: peer0.servicer-net:7051 # Internal Address of the other peer in same Org for gossip, same peer if there is only one peer          
          peerAddress: peer0.servicer-net:7051
          cli: enabled
          grpc:
            port: 7051
          events:
            port: 7053
          couchdb:
            port: 5984
          restserver:
            targetPort: 20001
            port: 20001 
          expressapi:
            targetPort: 3000
            port: 3000
          chaincode:
            name: "fabcar" #This has to be replaced with the name of the chaincode
            version: "1" #This has to be replaced with the version of the chaincode
            maindirectory: "fabric-sample/chaincode"  #The main directory where chaincode is needed to be placed
            lang: "golang"  # The language in which the chaincode is written ( golang/java/node )
            repository:
              username: "sivaramsk"          # Git Service user who has rights to check-in in all branches
              password: "53c9765ad883b0ab50ff7ca16f8b877597581de6"
              url: "git@github.com:sivaramsk/fabric-samples.git"
              branch: develop
              path: "fabcar/go/"   #The path to the chaincode 
            arguments: '{Make: "Toyota", Model: "Prius", Colour: "blue", Owner: "Tomoko"}' #Arguments to be passed along with the chaincode parameters
            endorsements: "" #Endorsements (if any) provided along with the chaincode  

~~

 Once everything is done, run 

docker run -it -v $(pwd):/home/blockchain-automation-framework/ hyperledgerlabs/baf-build bash

 From blockchain-automation-framework dir, run ansible-playbook -v platforms/shared/configuration/site.yaml  -e "@./build/network.yaml"  -e 'ansible_python_interpreter=/usr/bin/python3'

Create the cluster cheat sheet or Azure Cluster Details XL sheet and update

kubectl get ingress --all-namespaces
kubectl get service -n ingress-controller

#### DEPLOYING INTAIN COMPONENTS

Steps to be followed to deploy intain node app components:

1) Create a namespace called “fabricclient”
kubectl create namespace fabricclient

2) Create a fileshare in azure in the required storage account.

3) Create a secret named “azure-secret” by executing the create_secret_fileshare.sh script attached in the mail. Make sure you have entered the correct resource group, storage account and storage account keys in the script before you run.
//Refer umb google drive

```
STORAGE_KEY=$(az storage account keys list --resource-group <resource_group_name> --account-name <storage_account_name> --query "[0].value" -o tsv)

kubectl -n <namespace> create secret generic azure-secret --from-literal=azurestorageaccountname=<storage_account_name> --from-literal=azurestorageaccountkey=<key>

```
Create a file named create_secret_fileshare.sh and paste the above contents and update the details

4) Deploy mongodb using the following command,

helm install mongodb --set mongodbRootPassword=password,mongodbPassword=password stable/mongodb -n fabricclient
Once the mongodb pod started running, deploy the node internal app.

5) Create a secret “regcred” which will have our private docker registry credentials. Command to create the secret,

kubectl create secret docker-registry regcred  --docker-username=intainregistry --docker-password=7ddf9dbc-ee73-43cc-b77b-2d30809b3d4a --docker-email=sivaram.kannan@intainft.com -n fabricclient

6) Create storageclasses ( with azuredisk provisioner) using the shared azuredisk-sc.yaml 

allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fabricclient-disksc
parameters:
  kind: Managed
  storageaccounttype: Standard_LRS
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain
volumeBindingMode: Immediate

7) For internal app, we’ll be using azurefile as underlying storage. To deploy node internal app, create a pv with the created fileshare and create a pvc, deployment, service using the yaml files shared in the github repo

8) For external app, we’ll be using the disk as underlying storage. To deploy node external app, create a pvc, deployment and service using the yaml files in the shared github repo.

9) For ui, we’ll be using the disk as underlying storage. To deploy ui, create a pvc, deployment and service using the yaml files in the shared github repo.

10) Ingress has to be created for UI (Mandatory) and internal app (if needed). Before creating the ingress, install cert-manager and cluster-issuer in your cluster using the deployment yamls shared in the same github repo.

cd baf-intain-fork/platforms/hyperledger-fabric/configuration/build/
kubectl cp crypto-config/ <node-internal-app-pod>:/mnt/fabric -n fabricclient

To all the cli pods
kubectl cp crypto-config/ <pod name>:/etc/hyperledger/fabric -n fabricclient


 







