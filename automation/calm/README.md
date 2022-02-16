### 01/31/22 Actual information
We are switching to a new deployment strategy which only uses a single blueprint to make the deployment even more simple.
Find the old 2-Stage BPs in this Folder:
[a relative link](2-stageBP/README.md)

IMPORTANT: This new blueprint uses an Openstack image which is not supported by RedHat on Nutanix. Using this blueprint will deploy a cluster which is not supported by RedHat. This will change as soon there is a dedicated Nutanix image.
If you need a RedHat-supported cluster use the 2-Stage BP as this is based on the Platform-agnostic-Install Process.


## Using the blueprint
This blueprint is used to deploy an OpenShift cluster on Nutanix AHV.

Note that creating a cluster has the following requirements:
  - IPAM-enabled subnet with internet access and a configured IP-Pool for the VMs.
  - Static IP for our Bastion (which runs HAProxy and dnsmasq), which must be outside of the IP-pool.
  - existing DNS-Server (with the option to create a DNS-delegation for the subdomain we are creating)
  - access to the RHCOS image (this is only available .gz compressed and needs to be extracted and imported before running the blueprint)

## Caution
This blueprint uses an RHCOS-Openstack Base Image. Running Openstack-Images on any platform is not supported, so do not use this in a production environment. With upcomming Openshift 4.10 there will be a dedicated Nutanix image.
    
### Prequisite: DNS settings
Before running the blueprint please create a NS-Record or DNS-delegation for the desired OCP-subdomain. The delegation target-IP is used as Bastion-IP in our blueprint.

For example when using Microsoft DNS run a command like this:

```
Add-DnsServerZoneDelegation -Name "ntnxlab.local" -Nameserver "lbdns-ocp1.ntnxlab.local" -ChildZoneName "ocp1" -IPAddress 10.42.26.11 -PassThru -Verbose

```

### Prequisite: RHCOS image
Please import the following QCOW2 image into the Nutanix AHV cluster :
RHCOS OpenStack image (ex: https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.9/latest/rhcos-4.9.0-x86_64-openstack.x86_64.qcow2.gz)
If you import it manually please assign the image to the three services (bootstrap, master, worker).
As an alternative you can provide access to the image via HTTP and configure the URI in CONFIGURATION->DOWNLOADABLE IMAGE CONFIGURATION->RHCOS->Source URI
(Important: the pre-configured path needs to be changed when running the blueprint outside of NTNX-Environments)
### Customization (HAProxy/DNS)
By default a HAProxy is installed on LB_DNS-Service into which the services register (and are removed when doing a ScaleIn). You can replace the code inside of these actions if you want to use RestAPI against somekind of 3rd party loadbalancer for example.
There are also actions to register/remove DNS-Entries which can be modified to fit into your environment.

### Getting Started
1. Import  the blueprint
2. Within the blueprint select a valid subnet for all VMs
3. Store a RSA-private key within CREDENTIALS->CRED->Private Key
4. Deploy the blueprint as a new app
5. provide at least the following start-parameters:
   - IP of your Bastion VM (this is the target-IP from your DNS-delegation)
   - Number of workers
   - OCP_PULL_SECRET
   - BASE_DOMAIN
   - OCP_SUBDOMAIN (lower-letter, DNS-compliant)
   - OCP_MACHINE_NETWORK
 
## Logging into the Openshift console
After succesfull deployment, the auto-created login-information are accessible via Audit-Log->Create->OS_Status_Check Start->Show Login Information

  **Note: To use the web-console your client needs to use the matching DNS-server, otherwise create hosts-entries like this:
  a.b.c.d oauth-openshift.apps.SUBDOMAIN.BASEDOMAIN
  a.b.c.d console-openshift-console.apps.SUBDOMAIN.BASEDOMAIN

## Install the CSI-drivers
The stage 2 blueprint offers an action to deploy the Nutanix CSI Drivers. While running this action, additonal information like the Prism Element IP and credentials are collected.

## Configure image registry
This enables the Openshift image registry on a 100G Nutanix Volumes volume. While running this action, additonal Information like the dataservices IP, and the Nutanix storage container are collected.

  **Note: Block storage volumes like Nutanix Volumes with ReadWriteOnce configuration are supported but not recommended for use with the image registry on production clusters. An installation where the registry is configured on block storage is not highly available because the registry cannot have more than one replica.
