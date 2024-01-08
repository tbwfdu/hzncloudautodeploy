
# HznCloudAutoDeploy - Horizon Cloud NextGen Automation Tool
Deploy a Horizon Cloud on Azure Environment in under 5 minutes.

# Overview
HznCloudAutoDeploy is a console application that allows you to validate and automate a Horizon Cloud on Azure Deploymment. This tool will create and deploy all of the necessary resources and pre-requisites into your Azure Subscription to same supported configuration of the [Horizon Cloud Azure Pre-Requisites](https://docs.vmware.com/en/VMware-Horizon-Cloud-Service---next-gen/services/hzncloud.nextgen/GUID-63AB2398-E82F-4EF2-BC7F-145D79E28769.html).

This is by far the most common usage scenario for this tool.

It also possible to automate the configuration steps in the Horizon Cloud NextGen Console using HznCloudAutoDeploy. It will configure the infrastructure that was previously provisioned in your Azure environment as well as any Edge Appliances and Unified Access Gateways, along with the required Active Directory configurations.

The application is written in .NET Core and compiled binaries for **Windows** and **macOS** are available.

See below for configuration items and descriptions.

# High Level Capabilities 
HznCloudAutoDeploy allows you to initially validate an Azure Subscription for pre-existing configurations that are in conflict with the configuration items in the deployment section of this application. If there are no issues, the application will read the parameters in the two (2) YAML files from the same directory and proceed to deploy and configure your Azure Infrastructure to meet the Horizon Cloud NextGen requirements. Similarly, if you choose to automate the Horizon Cloud components, it will check for conflicts and then proceed to deploy the configurations.

**_The Application Will:_**

**Azure:**

- Create a Single Resouce Group in your Azure subscription.
- Create a VNet in this Resource Group.
- Creates three (3) Subnets in this VNet:
- Management Subnet
- DMZ Subnet
- Desktop Subnet
- Create a Public IP Prefix (reservation)
- Create a Public IP for a NAT Gateway
- Create a NAT Gateway
- Assign the NAT Gateway to the Management Subnet
- Assigns the DNS Server settings to the VNet.
- Ensures that all required Resource Providers are registered in Azure.
- Creates two (2) Custom Roles:
  - Service Principal Role with minimum capabilities needed for the Service Principal used by Horizon Cloud.
  - Azure Compute Read-Only role with permissions on to Read on Azure Compute Resources.
- Creates a User Managed Identity
- Assigns the Managed Identity the Network Contributor & Managed Identity Operator built-in roles.
- Creates an Enterprise Application.
- Creates a Service Principal, ClientID and Client Secret for the Enterprise Application.
  
All results/output of each part of this deployment process is output (in .JSON) to a ./results folder in the same executable directory.

**Horizon Cloud:**

- Add an Active Directory Configuration
- Create a SSO Configuration for your Domain
- Create a Site
- Create an Azure Provider for your Azure Subscription
- Deploy an Edge Cluster using Azure AKS, to your Azure Subscription
- Create a Site to Edge Mapping
- Deploy a UAG Cluster to your Azure Subscription

This application also has the **ability to delete** (cleanup) an environment that has been created using the tool.

For Horizon Configurations deployed using HznCloudAutoDeploy it will:

- Delete the UAG Cluster (waits for confirmation of deletion)
- Delete the Edge Cluster (waits for confirmation of deletion)
- Deletes the Site to Edge Mapping
- Deletes the Site Configuration
- Deletes the SSO Configuration
- Deletes the Active Directory Configuration.

And, then if the above has been completed successfully:

For Azure Configurations deployed using HznCloudAutoDeploy it will:

- Delete the NAT Gateway
- Delete the Resource Group
  - This deletes the VNet and Subnets
- Delete the IP Address Prefix
- Delete the NAT Gateway Public IP
- Delete the Managed Identity
- Delete the Service Principal
- Delete the Custom Role Assignments
- Delete the Custom Roles

After completing this cleanup it should delete all resources that been configured in Azure as well as any Horizon Configurations.

> [!IMPORTANT]
>Deleting the UAGs, Edges and Azure Provider from Horizon Cloud does not delete the Azure File Share that was provisioned for App Volumes. If you are sure you no longer want to keep these, you need to delete them manually.

> [!NOTE]
> If you need to locate the Client ID and Secret for the Service Principal, this can be found in ./results/servicePrincipal/secret.json

# Downloads
The latest version of HznCloudAutoDeploy is [available here](https://github.com/tbwfdu/hzncloudautodeploy/releases/tag/v1.3)

# Usage  

**Windows:** 

  Download the correct file executable, along with the config.yaml and the params.yaml file.These YAML files must be placed in the same directory as the exectuable.
Double click the file to open automatically in a Command Line window.

**macOS:** 

Download the correct file executable, along with the config.yaml and the params.yaml file. These YAML files must be placed in the same directory as the exectuable.
Open the Terminal app, navigate to the location you downloaded the Application to, and run      ./HznCloudAutoDeploy      (double clicking to execute runs it from your home folder and will not locate the YAML files).


**Usage Notes:**

Both Deployment processes will output their results to a folder under another folder called results. Horizon Deployment uses these results to provision the infrastructure and configurations to the CSP (eg. it will read the vnet.json file under .\results\networks to add the network config).

By default, when you exit the application (using the exit option in the menu) it will create a .zip file of the contents of the results folder, and then delete the results folder. If you want to deploy Horizon after previously successfully deploying Azure (or closing the app) you can either use the "Extract Most Recent Results" option which will automatically look for an extract the latest results*.zip archive, or you can open the results*.zip archive with the successful deployment results in it, extract the single file called "azureResults.json" and place it in the results folder (create the folder if needed). Then, you can use the "Use Existing Deployment Results" option, and then deploy Horizon.

> [!IMPORTANT]
> Before you can deploy either Azure or Horizon, you must run a pre-deployment validation. Any errors or warnings will not stop you from deploying, it just ensures you know what issues there are.

# Configuration Parameters

> [!IMPORTANT]
> User Editable Parameters are in config.yaml. The entries in params.yaml are for configuring roles and resource providers as specified by VMware Documentation and **should not** be adjusted.


**Configuration Parameters** **(_all mandatory and non-empty_)**:
> [!NOTE]
> YAML is specific about indentation and spacing - ensure the existing formatting is kept)



(Example contents of config.yaml file):

```YAML
azure:
  subscription_id: "c3a9527a-b33d-42e9-ac60-3b02ea6700fd"
  tenant_id: "ea5fd243-1e94-4d96-ac60-21d9e53fcb0c"
  cache_auth: "true"
resource_group:
  name: "HznCloudAutoDeploy"
  deployment_location: "australiaeast"
vnet:
  name: "hzn_cloud_vnet"
management_subnet:
  name: "hzn_mgmt_subnet"
  range: "192.168.252.0"
  subnet: /27
dmz_subnet:
  name: "hzn_dmz_subnet"
  range: "192.168.253.0"
  subnet: /27
desktop_subnet:
  name: "hzn_desktop_subnet"
  range: "192.168.254.0"
  subnet: /24
dns_servers:
  - "x.x.x.x"
  - "y.y.y.y"
  - "168.63.129.16" # <------ DO NOT REMOVE THIS ENTRY
nat_gateway:
  name: "hzn_cloud_nat_gateway"
aks_edge_cluster:
  docker_bridge_cidr: "172.17.0.0/16"
  service_cidr: "192.168.251.0/27"
  pod_cidr: "192.168.240.0/21"
  nodes: "4"
azure_public_ip:
  prefix: "29"
  prefix_name: "hzn_cloud_pub_ip_prefix"
  ip_name: "hzn_cloud_pub_ip"
service_principal_role_name: "Horizon Cloud Custom Service Principal Role"
service_principal_role_description: "All permissions required for deployment and operation of a Horizon Edge in Azure."
service_principal_name: "HznCloudServicePrincipal"
compute_ro_role_name: "Horizon Cloud Azure Compute Read-Only Role"
compute_ro_role_description: "Custom Read-Only Role for Microsoft Azure Compute Galleries."
managed_identity_name: "HznCloudManagedIdentity"
horizon:
  csp_api_key: "xxxxxxxxxxxxxxxxxxxx"
  csp_org_id: "ea5fd243-1e94-4d96-ac60-21d9e53fcb0c"
active_directory:
  name: "YOURCOMPANY"
  fqdn: "your.company.com"
  bind_primary_username: "bindadmpri"
  bind_primary_password: "password"
  bind_aux_username: "bindadminaux"
  bind_aux_password: "password"
  join_primary_username: "joinadminpro"
  join_primary_password: "password"
  join_aux_username: "joinadminaux"
  join_aux_password: "password"
  default_ou: "OU=Computers,DC=company,DC=local"
ca:
  config_dn: "CN=Configuration,DC=company,DC=local"
  ca_mode: "root"
sites:
  site_name: "HznCloudAutoDeploySite"
edges:
  edge_name: "HznCloudAutoDeployEdge"
  enable_private_endpoint: "true"
  edge_fqdn: "edge.company.com"
uags:
  cluster_min: "1"
  cluster_max: "2"
  uag_name: "HznCloudAutoDeployUAG"
  uag_fqdn: "uag.company.com"
  number_of_gateways: "2"
  type: "INTERNAL_AND_EXTERNAL"
ssl_certificate:
  cert: ""
  password: ""
  type: "PEM"
cleanup_results: "true"
```



**Configuration Notes:**

_Azure:_

**Customer Subscription ID** - The ID of the Subscription in Azure you want to deploy against.
```
subscription_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```
**Customer Tenant ID** - The Tenant ID in Azure
```
tenant_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```
**Cache_Auth** - Cache Azure Authentication tokens (recommended).
```
cache_auth = "true" or "false"
```
**Resource Group:**

**Name** - The name of the Resource Group that will be created in Azure.
```
name = "HznCloudAutoDeploy" (can be adjusted)
```

**Azure Region** - Where you want to create the Resource Group and deploy all Horizon Components.
```
deployment_location = "australiaeast" (can be adjusted)
```
**Network**:

**VNets and Subnets** - Details of the VNet and Subnets required
```
management_subnet = "192.168.252.0/27" - Management (Edge) VNet Range
(can be adjusted, minimum of /27, must not conflict refer to documentation)
```
```
dmz_subnet = "192.168.253.0/27" - DMZ VNet (UAG) Range
(can be adjusted, minimum of /27, must not conflict refer to documentation)
```
```
desktop_subnet = "192.168.254.0/24"  - Desktop (VM) Range
(can be adjusted, must not conflict refer to documentation)
```
**Customer Domain DNS Server Address & External DNS** - Should be internal DNS for internal resource resolution, plus added external for external resolution
```
dns_servers:
  -8.8.8.8
  -1.1.1.1
```

**Docker, Service and Pod CIDR** - Used for Edge Cluster AKS deployment, these will be in their own Resource Group (and a new VNet will be created for these subnets).

**Roles & Account Names**

Name of the custom role to be created in place of Contributor for Horizon Cloud Service Principals
```
service_principal_role_name = "Horizon Cloud Custom Service Principal Role" (can be adjusted)
```
Service Principal Name to be created
```
service_principal_name = "HznCloudServicePrincipal" (can be adjusted)
```
Name of the custom role to be created for Azure Compute ReadOnly
```
compute_ro_role_name = "Horizon Cloud Azure Compute Read-Only Role" (can be adjusted)
```
Managed User Identity Name to be created
```
managed_identity_name = "HznCloudManagedIdentity" (can be adjusted)
```


**Horizon:**

**CSP API Key** - In your Cloud Services Portal, under your User Account, go to API Keys and generate an API Key.

**CSP API Key** - In your Cloud Services Portal, copy your Organization ID (long version) by clicking your name in the top right corner, and it will appear at the top of the menu.

**Active Directory** -

Name is a "friendly name" to reference your dir by.

FQDN is your directory's internal dns name (eg. company.local or company.com etc.)

Bind & Join Primary and Auxilliary accounts. These are used for connecting to AD and Joining VMs to AD. Refer to documentation for more information.

> [!IMPORTANT]
> SSL Certificate - Currently this application will only use the PEM format, and the filename of your certificate must be located at the root of the HznCloudAutoDeploy application and must be named cert.pem

# Screenshots

| Example Screenshots | Description |
| --- | --- |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-igb.png) | Main Menu |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-ii8.png) | By default, it will use a Browser to get Azure Authentication |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-ijj.png) | View Azure Configuration Details |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-ihk.png) | View Horizon Configuration Details |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-ij0.png) | Azure Deployment Validation Warnings |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-ij6.png) | Azure Warnings Detailed Information |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-ijy.png) | Horizon Cloud Detailed Warnings |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-ih8.png) | Pre-Deployment Confirmation |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-ikh.png) | Azure Pre-Deletion Confirmation |
| ![screenshot](https://github.com/tbwfdu/hzncloudautodeploy/blob/main/images/SCR-20230222-il0.png) | Horizon Pre-Deletion Confirmation |
