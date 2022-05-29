# Azure Viya Partner Env IAC
[Link](https://github.com/andrewGreggSAS/azure-viya-partner-sandpit-iac)

Full Infrastructure and software deployment orchestration for SAS Viya 4 on Azure K8s.
Use-case: Create a dedicated Azure K8s deployment of SAS Viya 4 for the purposes of a Partner Sandpit. Capably sized and deployable in 24 hours.

If you haven't already done so watch the deployment overview viya4 deployment video https://video.sas.com/sharing?videoId=6204768249001
this will give an overall idea of the the intent of the deployment and the steps we are automating

## Summary

The aim of the Azure Viya Partner Env IAC is to enable fast deployment of SAS Viya (4) deployed on Azure Kubernetes Service (AKS).
This deployment can be used in support of Viya demonstrations or as a sandpit to experiment and test new Viya Capabilities and features

This project has utilised the existing automation in these SAS Github Repositories:

https://github.com/sassoftware/viya4-iac-azure

https://github.com/sassoftware/viya4-deployment

Note: As changes are made and improvements/features are added,  they may or may not seamlessly merge. Work to include new components that require configuration may continue in the future if there is value seen in doing so.

The default config files deploy the the environment with the following configuration:

* Multiple node pools (stateless, stateful, cas, compute, connect)
* Default Internal “Crunchy” PostgreSQL DB
* OpenLDAP for independent management of all users and groups ( not working)
* NFS Server for application and user (home) data

Variables files allow you to vary the specification of the infrastructure easily (see below)

## Getting Started
### Set Up Prerequisites Summary

* You must have an Azure subscription to which you (your own Azure login) have authority to create Service Principals with Contributor roles, being an Owner of your own Azure subscription is the target use-case here

* You must have a Viya 4 order that is at a minimum SAS Visual Data Science preferably SAS Visual Data Science Decisioning

* You must have access to your order at https://my.sas.com/en/my-orders.html

* You will need to create and then fetch the SAS Order API Key and client_secret

* You must create an Ubuntu machine connected to your network
    * One way to achieve this is to utilise the latest Windows Subsystem for Linux (WSL2) release of Ubuntu
    * Open the Microsoft Store on your SAS laptop
    * Type ‘Ubuntu 20.04 LTS’ in the search bar
    * Click the blue ‘Get’ button
    * Read the description and follow the steps to enable required the Windows feature

    * Another way is Linux VM somewhere within a virtualised container software like Virtual Box. Again install Ubuntu 20.04 on it

    * When installing Ubuntu in your virtual environment make sure you set up a user in addition to root

### Detailed Pre-requisites
#### Azure subscription
* There are resources that need to be increased to cater for the viya deployment
    1. Increase Standard Dv4 Family vCPUs to 350 from 10
    2. Increase Standard EDSv4 Family VCPUs to 350 from 10
    3. Increase Total Regional vCPUs(Cores-vCPUs) 410 from 10
#### SAS Viya Order
* You need permission to download the viya Order
  * If you haven't already done so watch the deployment overview viya4 deployment video https://video.sas.com/sharing?videoId=6204768249001
  this will give an overall idea of the steps we are automating
  [insert screenshot and link to PNG on how to do this]
  * steps
    * Open the email with your Viya or or access https://my.sas.com/en/my-orders.html
    * See SAS Installation Documentation https://documentation.sas.com/?docsetVersion=v_001&docsetId=mysasprtl&docsetTarget=p0tu7dceilgd9mn17o4p6j674nz9.htm#p0ne79e56scnp0n1scgg9yilenaf

* Create the SAS Viya Order Api ( If you have not already done so)
  [insert screenshots here]
  * steps
    * From the My Orders window open the tool icon and select Viya Orders API
      *Create you Viya Order Api - If you have done this you can skip this steps
      [insert screenshots here]
* Copy the SAS Order Api Key and client_secret
  [insert screenshots here]
      Open the API you created in the step above
      copy the API Key and Client Secret. Store these values in a safe place as they will be used in the deployment_repo-variables.yaml step later

## What this package is NOT and never will be
Note: the intent of this package is to get a Viya 4 environment up and running quickly. HOWEVER this is not meant to replace the wonderful Viya Installation and Viya Administration enablement pathway available for partners in PartnerNet

## Documentation
As always refer to SAS online Documentation
Viya Deployment https://go.documentation.sas.com/doc/en/itopscdc/v_025/dplyml0phy0dkr/titlepage.htm

Viya Administration https://go.documentation.sas.com/doc/en/sasadmincdc/v_025/sasadminwlcm/home.htm


## Deployment

### Step 1 - Clone
Log into your Ubuntu 20.04 installation machine and clone this Repo:
```
git clone https://github.com/andrewGreggSAS/azure-viya-partner-sandpit-iac
```
### Step 2 - Edit
There are 4 configuration (YAML) files to edit
1. *core-variables.yaml* MUST be edited with your own details,
  * your Azure subscription details and
  * two variables you need to choose:
    {deployment_name} - The label stem for many of the infrastructure and resources
    {deployment_environment} - which will be environment name for THIS Viya 4 deployment and the name of the kubernetes namespace for it.

It is IMPORTANT that BOTH of these variables conform to the following rules:
  * Eleven (11) characters or less so that all the derived fields have names that are not too long.
  * start with a lower-case letter
  * only contain letters, numbers and dash (-)
  * must not end with a dash

```
cd azure-viya-partner-sandpit-iac
nano core-variables.yaml
```

When your deployment is complete, the environment will be accessed on the URL {deployment_environment}-{deployment_name}.{azure_location}.cloudapp.azure.com. So as an example, I use partner as my *deployment_name* and a simple short name indicating the Viya environment's purpose like *sandpit*, making the URL:
    *sandpit-partner.australiaeast.cloudapp.azure.com*

This will allow deploying multiple environments to a single set of kubernetes infrastructure, as well as deploying other adjacent services to explore integration.

2. *resources/deployment_repo-variables.yaml* MUST be edited with your own software order information.

These should have been stored following the instructions in Copy the SAS Order Api Key and client_secret above

there are 2 other version filds that *may* be edited
  * iacazure:tag: corresponds the *version* of SAS's Infrastructure as Code GIT repository you would like to used see https://github.com/sassoftware/viya4-iac-azure

  * viya4deployment:versiontag: corresponds to the SAS *version* of viya deployment GIT repository https://github.com/sassoftware/viya4-deployment you wish to use

```
nano resources/deployment_repo-variables.yaml
```

All other variables files contain current tested default values. These can be customized to your requirement but changes to client software versions (such as ansible, kubectl or terraform) can cause issues that will need manual troubleshooting and intervention.

3. *iac/iac_build-variables.yaml* MUST be edited to update the variable
network:defaultpublicaccesscidrs - this can be your home machine IP or work network. this uses CIDR notation. there may also be multiple IPs each seperated by a comma
```
nano iac/iac_build-variables.yaml
```

4. *viya/viya-deployment-variables.yaml* MUST be edited to update the variable ingress:sourceranges - This should be same list of CIDRS as specified in the iac/iac_build-variables.yaml file for network:defaultpublicaccesscidrs
```
nano viya/viya-deployment-variables.yaml
```

#### Step 3 - Run scripts in sequence
There are four scripts that each must be run  to complete the process, each with its own folder also containing a variables yaml file.


1. ubuntu_deployer - Prepares your local machine with packages and configuration required
```
./ubuntu_deployer/ubuntu_deployer-setup.sh
```
2. resources - Retrieves all the required resources from other git repos and download locations and sets them up on your local machine
```
./resources/deployment_repo-setup.bash
```
3. iac - Creates the specified infrastructure in your Azure subscription to support the SAS Software deployment (using Terraform)
```
./iac/iac_build.bash
```
4. viya - Deploys the SAS software (With Kustomize and kubectl)
```
./viya/viya-deployment.bash
```

### Done!

Well, YOU are at least. The system itself is working hard still..
You should get a message once deployment is complete with a URL and credentials to log in. However, your SAS pods are still getting scheduled and starting up on the set of node pools and this will still take some time to complete. Each of the nodes must download the container images from the SAS repository for each of the pods it's running. Bandwidth limits the speed that this can occur, so although you've done your bit the deployment still has a fair amount to work through.

## Troubleshooting

In general the scripts provide meaningful error messaging. If a step clearly fails to complete successfully in the execution of a script, it is unlikely that subsequent steps will succeed. Read the error message, understand what it is saying and what could be causing the problem and try to take corrective action, then run the failed script again.

The aim is to build tasks in each script that are idempotent, such that re-running would not break anything. However strictly adhering to this ideal is an ongoing challenge.


## This is where the Automation Ends...

Now that you have a Viya deployment, you need to look after it. I've put a few scripts and snippets here that I use to help you out:

#### Starting and stopping

The following commands pause your kubernetes cluster. with the following notes:
* The cluster state of a stopped AKS cluster is preserved for up to 12 months. If your cluster is stopped for more than 12 months, the cluster state cannot be recovered. For more information, see the AKS Support Policies.
* You can only start or delete a stopped AKS cluster. To perform any operation like scale or upgrade, start your cluster first.
* The customer provisioned PrivateEndpoints linked to private cluster need to be deleted and recreated again when you start a stopped AKS cluster.”


The following is based on a SAS communities article found HERE
https://communities.sas.com/t5/SAS-Communities-Library/Reduce-the-Azure-bill-when-you-are-not-using-the-SAS-Viya/ta-p/789920 see the Section: Implementing the AKS stop/start with Viya

Use az aks stop command to pause the cluster

```
az aks stop --name ${deployment_name}-aks --resource-group ${deployment_name}-rg --subscription ${subscription}

```
Use az aks start command to un-pause the cluster

```
az aks start --name ${deployment_name}-aks --resource-group ${deployment_name}-rg --subscription ${subscription}

```

#### Shell Environment

```
source ~/pyvenv_ssaima/bin/activate
```
(This is the example for my environment, substitute ssaima with your $deployment_name.)
A lot of the client tools use python and the cleanest way to maintain dependencies is to use a virtual environment. One has been set up during the deployment and it is for THIS deployment specifically (has all the right versions). Whenever you log on to administer this environment you should first activate  using the binary

```
source $HOME/azure-viya-ca-env-iac/source_all.bash
```
This simply sources all the variables used in the deployment for use in your current shell. Very helpful if you have multiple environments as you can use shell parameterized commands with automatic substitution. Below are some of these examples.

#### Watch the deployment start
```
kubectl get pods -o wide -n ${deployment_environment}
```
See all the pods starting up and look for issues and errors.
#### Getting your credentials for applying changes
Before making changes there are 4 terraform variables that need to be set
These variables are set in the  ${deployment_name}.tfvars file in the
Azure Auth Section
uncomment and quote the values for the following
 tenant_id = ""
 subscription_id = ""
 client_id = ""
 client_secret = ""

Make changes in the ${deployment_name}.tfvars file
```
cd ~/${deployment_name}-aks/viya4-iac-azure
nano ${deployment_name}.tfvars
```
They are

#### Making changes

The artifacts, repos and configuration written by the four scripts is placed in your home folder named *$(deployment_name)-aks*
```
cd ~/$(deployment_name)-aks
```
Inside this folder there are many folders and files now. The *viya4-iac-azure* folder is the cloned repo for the sassoftware github Infrastructure-As-Code, with edits made specific to your deployment. Here you can add, edit and apply changes using terraform as you might with any other Viya deployment.

Re-generate the Terraform plan after making changes to the infrastructure specification (eg. adding more nodes to a pool)
```
# Don't forget to setup your shell with python virtual env and shell environment variables
source ~/${deployment_name}/bin/activate
source $HOME/azure-viya-partner-sandpit-iac/source_all.bash

cd $HOME/${deployment_name}-aks/viya4-iac-azure

# DO SOME CHANGES IN HERE THAT NEED A NEW TERRAFORM PLAN
#   eg. I might edit ssaima.tfvars to change the cas node_pool to have "max_nodes" = "4" for added capacity

# Create the updated terraform plan
terraform plan -input=false \
    -var-file=./${deployment_name}.tfvars \
    -out ./${deployment_name}-aks.plan

cd $HOME/${deployment_name}-aks/viya4-iac-azure
TFPLAN=${deployment_name}-aks.plan

# Apply the terraform plan
time terraform apply "./${TFPLAN}" 2>&1 \
| tee -a $HOME/${deployment_name}-aks/viya4-iac-azure/$(date +%Y%m%dT%H%M%S)_terraform-apply.log

# After this you MAY need to regenerate your kube_config file
terraform output kube_config | sed '1d;$d' > ~/.kube/${deployment_name}-aks-kubeconfig.confcd
```

The $deployment_environment folder (eg. *proving* for my example) contains the kubernetes configuration for your Viya software environment. In here you can add and remove resources, configurations, transformers, generators and components into the site-config folder, and corresponding references in kustomization.yaml to change the kubernetes deployment.
There is much more detail about how this all works in the SAS Viya Operations Guide for your chosen software version. Additionally, read up on how this folder was generated using viya4-deployment at the [viya4-deployment sassoftware Github page](https://github.com/sassoftware/viya4-deployment)

```
cd $HOME/${deployment_name}-aks/${deployment_environment}/

# DO SOME CONFIG CHANGES IN HERE

kustomize build -o site.yaml

kubectl apply -f site.yaml

```

## Destroy Everything!!

So you're done and now you need to get rid of it all? Forever? Irretrievably?
Simple, you just need to create a terraform "destroy" plan:

```
# Generate the DESTROY plan
cd $HOME/${deployment_name}-aks/viya4-iac-azure
terraform plan -input=false \
    -destroy \
    -var-file=./$deployment_name.tfvars \
    -out ./$deployment_name-aks-destroy.plan

# Run the DESTROY plan
echo "[INFO] Destroying the AKS cluster infra"
time terraform apply $deployment_name-aks-destroy.plan 2>&1 \
| tee -a $HOME/${deployment_name}-aks/viya4-iac-azure/$(date +%Y%m%dT%H%M%S)_terraform-destroy.log
```

All gone!

(If not you can always log into portal.azure.com and delete the two resource groups that bear your ${deployment_name})


## Authors

[Andrew Gregg] (mailto:Andrew.Gregg@sas.com)

## Thanks to

Isaac Marsh

## Version History
