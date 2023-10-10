# FlashStack VSI CVD using Cisco UCS M7 servers, Pure Storage FlashArray, and VMware vSphere 8.0 

The FlashStack portfolio of solutions are a series of validated solutions developed in partnership with Cisco and Pure Storage. Each FlashStack is designed, built, and validated in Cisco’s internal labs, and delivered as Cisco Validated Design (CVD) that includes a design guide, deployment guide and automation delivered as Infrastructure as Code (IaC).

This release of the CVD introduces support for the 7th generation of Cisco UCS C-Series and Cisco UCS X-Series Servers, powered using 4th Gen Intel Xeon Scalable processors. The new Cisco UCS servers provide the compute infrastructure for the FlashStack virtual server infrastructure (VSI) solution. For storage, the solution uses either 32Gbps Fibre Channel (FC) or 100Gbps IP/ethernet storage to access unified block and file storage on a Pure Storage FlashArray. For the virtualization layer of the stack, the solution introduces support for VMware vSphere 8.0 and the new capabilities available in this release. 

This repository provides the IaC automation for provisioning Cisco, Pure Storage and VMware components in this CVD solution:
https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/UCS_CVDs/flashstack_m7_vmware_8_ufs_fc.html

This repository contains the Ansible playbooks to automate the following components in the Virtual Server Infrastructure stack: <br />
&emsp;&emsp; •	 Cisco UCS X-series and C-series M7 servers in Intersight Managed Mode (IMM) <br />
&emsp;&emsp; •	 Cisco Nexus and MDS Switches <br />
&emsp;&emsp; •	 Pure FlashArray   <br />
&emsp;&emsp; •	 VMware ESXi and VMware vCenter.  <br />

The automation provided in this repo will enable you to deploy the following FlashStack designs in the CVD lusing Ansible. To execute the playbooks, the following FlashStack solution design must be built and cabled as shown in the figure below. The automation does provide the flexibility to provision either FC or IP/Ethernet for storage access.

## FlashStack - Solution Topology

![image](https://github.com/ucs-compute-solutions/FlashStack_IMM_M7/assets/24396268/960946fd-2feb-4795-9781-7d1579560e01)
<br />

## Solution Automation Overview
<img width="1200" alt="Screen Shot 2022-09-15 at 5 03 58 PM" src="https://user-images.githubusercontent.com/25094641/190393678-6cafd996-fae2-4553-8f23-12cbe1c11293.png">

<br />

### Set up the execution environment
To execute various ansible playbooks, a Linux based system will need to be setup as described in the CVD with the packages listed at the following pages: <br />
-	Cisco Intersight: https://galaxy.ansible.com/cisco/intersight <br />
-	Cisco NxOS: https://galaxy.ansible.com/cisco/nxos <br />
-	Pure FlashArray: https://galaxy.ansible.com/purestorage/flasharray <br />
-	VMware: https://galaxy.ansible.com/community/vmware <br />

You might already have this collection installed.

- To check whether it is installed, run: `ansible-galaxy collection list`
- To install it, use: <br />
  `ansible-galaxy collection install cisco.intersight` (For Intersight Collection) <br />
  `ansible-galaxy collection install cisco.nxos` (For Cisco NX-OS collection)  <br />
  `ansible-galaxy collection install purestorage.flasharray` (Pure Storage FlashArray Collection) <br />
  `ansible-galaxy collection install community.vmware` (For VMWare Ansible Collection) <br />

Next, clone the repository from Github with "git clone https://github.com/ucs-compute-solutions/FlashStack_IMM_M7.git".

<br />

### Setup Inventory and Variables

To execute the automation in this repo, update the inventory and variables files to reflect the specifics of your environment. The variables that must be configured to execute the automation are defined in the following locations:

1. Inventory file is located: inventory/inventory.ini file 
2. Variable are distributed across multiple files. 
&emsp;&emsp; •	Variables that require customer inputs are part of inventory/(group_vars|host_vars)/<component_name>/vars (and vault)
&emsp;&emsp; •	Variables that do not typically require customer input (e.g. descriptions etc.) are present under role_name/defauls/main.yml.
&emsp;&emsp; •	Intersight pools, policies and policies created using these playbooks will start with user_defined_prefix (for e.g. M7a) in the name and a tag of "Provisioned by: intersight-ansible" to quickly identify the ones created using Ansible.

<br />

### Prerequisites

1. To deploy UCS configuration via Intersight, you will need an Intersight account with the Cisco UCS Fabric Interconnects claimed and provisioned using a UCS domain profile. 
2. Collect the UCS host IQNs (IP/Ethernet) or WWPNs (FC) from the server profiles once they have been provisioned/derived from the server profile template. Update the variables files for Pure Storage and MDS switches with this information. 
3. Collect the target IQNs (IP/Ethernet) or WWPNs (FC) from Pure Storage once the ethernet/fc interfaces have been provisioned for the storage protocol/service. Update the variables files for Cisco UCS and MDS switches with this information. 
4. To deploy the vCenter and vSphere setup for the solution, you should have a VMware vCenter deployed. 
 
<br />

### Playbook Execution Sequence

**NOTE:** You can specify the location of the inventory file by adding `-i inventory/inventory.ini` to the ansible-playbook command below. 

1.	Setup LAN networking on a pair of Nexus access layer switches: `ansible-playbook ./setup_network.yml` 
2.	Setup Pools, Policies and Profiles for UCS through Cisco Intersight: `ansible-playbook ./setup_compute.yml`
3.	Setup Pure FlashArray: `ansible-playbook ./setup_pure_storage.yml`
4.	Setup FC SAN networking on a pair of MDS switches: `ansible-playbook ./setup_network_san.yml`
5.	Setup VMWare Cluster and vCenter Setup: `ansible-playbook ./setup_vmware_vcenter.yml`
6.	Setup VMWare ESXi servers: `ansible-playbook ./setup_vmware_esxi.yml`

<br />

### Intersight Configuration and Access Requirement

The Intersight playbooks in this repository perform following functions:

1. Create various pools required to setup a Server Profile Template
2. Create various policies required to setup a Server Profile Template
3. Create iSCSI and/or FC Server Profile Templates

After successfully executing the playbooks, one or many server profiles can easily derived and attached to the compute node from Intersight dashboard.

<br />

To execute the playbooks against your Intersight account, you need to complete following additional steps of creating an API key and saving the Secrets_File:

https://community.cisco.com/t5/data-center-and-cloud-documents/intersight-api-overview/ta-p/3651994

The API key and Secrets.txt (default file name) should be put in the inventory/intersight/vault sub-directory and encrypted - these as well as passwords should also not be uploaded to GitHub by using the .gitignore file to filter out any files with sensitive information. 

<br />

### Post Configuration Tasks

Execution of first three playbooks in these repositories set up Server Profile Template in Intersight. After successfully executing the playbooks, one or more server profiles can easily derived and attached to the compute node from Intersight dashboard. To install OS on these newly provisioned servers, Intersight now offers an Install OS workflow that can be directly accessed by navigating to **Infrastructure Service > Operate > Servers**. Next select a server and use the '...' option on the right hand side to select **Install Operating System"** from the list of menu options. 

<br />
