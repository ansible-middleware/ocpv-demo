# Ansible Middleware OpenShift Virtualization Demo

Demonstrating the use of OpenShift Virtualization as an environment for hosting Virtual Machines that are provisioned, configured and maintained using Ansible and tooling from the Ansible Middleware project.

## Environment Requirements

For this demonstration, the following resources must be available:

* OpenShift Container Platform
    * Capable of running OpenShift Virtualization ([Requirements](https://docs.openshift.com/container-platform/4.12/virt/install/preparing-cluster-for-virt.html#virt-hardware-os-requirements_preparing-cluster-for-virt))
    * `cluster-admin` access
* Ansible Controller (Part of Ansible Automation Platform) deployed to the same OpenShift environment as OpenShift Virtualization.
* Control Node capable of provisioning the demo environment
    * Ansible

## Prerequisites

Certain requirements must be met prior to realizing the full potential of this demonstration. This involves not only completing steps within a local machine (control host), but also obtaining assets from external systems.

### Clone the Repository

Tooling is availale within this repository to provision the demonstration. Clone the repository to your local machine:

```shell
git clone https://github.com/ansible-middleware/ocpv-demo.git
cd ocpv-demo
```

### SSH Keypair

An SSH keypair is needed to communicate with the provisioned virtual machine. Provide the location of an SSH (without password) public key (`.pub`) and private key within the `ssh_public_key_path` and `ssh_private_key_path` variables in the [vars/provision.yml](vars/provision.yml) file.

### Automation Hub Token

A token for the public Automation Hub must be obtained so that it can be configured to access certified content locally as part of the setup and provisioning process as well as being stored in Ansible Controller. 

Steps for obtaining the token can be found [here](https://cloud.redhat.com/ansible/automation-hub/token/)

The token must be added to two (2) locations:

1. Set the value of `token` in the [ansible.cfg](ansible.cfg) file
2. Set the value of the `automationhub_token` variables in the [vars/provision.yml](vars/provision.yml) file

### Red Hat Service Account

A Service Account as the associated Client ID and Client Secret must be provided so that the Runtimes assets can be generated from the Red Hat Customer Portal.

Obtain a Client ID and Client Seceet by generating a Service Account [here](https://console.redhat.com/application-services/service-accounts).

Set the values of the `jbossnetwork_client_id` and `jbossnetwork_client_secret` variables in the [vars/provision.yml](vars/provision.yml) file.

### Red Hat Customer Portal Account

The provisioned Virtual Machine will be subscribed so that it can obtain the required packages. Populate the `redhat_csp_username` and `redhat_csp_password` variables in the [vars/provision.yml](vars/provision.yml) file with these values.

### Ansible Controller

The provisioning process will configure Ansible Controller to manage the demonstration. The location of Controller as well as credentials must be specified. Set the `controller_hostname`, `controller_username` and `controller_password` variables in the [vars/provision.yml](vars/provision.yml) file.

### Python Dependencies

Install the required Python dependencies by executing the following command from the root of the cloned repository:

```shell
pip install -r requirements.txt
```

### Ansible Dependencies

Install the required Ansile dependencies by executing the following command from the root of the cloned repository:

```shell
ansible-galaxy collection install -r requirements.yml
```

## Provision the Demo

Provisioning the demonstration performs the following actions:

* Configures Ansible Controller
    * Custom Credential Types
    * Credentials
    * Organization
    * Project
    * Inventory
    * Job Templates
        * Deploy OCPv
        * Deploy Demo

Ensure that you are logged into the OpenShift environment with a user with `cluster-admin` permissions and execute the following command:

```shell
ansible-playbook playbooks/provision.yml
```

## Executing the Demo

Once the configuration of Ansible Controller has completed successfully, login to Ansible Controller and navigate to the **Templates** tab underneath _Resources_.

Deploy OpenShift Virtualization by selecting the rocketship next to the _Deploy OCPv_ Job Template to install and configure OpenShift Virtualization.

Once OpenShift Virtualization has been installed, execute the demo. The demo provisioning process will perform the following actions:

1. Create a new OpenShift namespace called `ansible-middleware-ocpv`.
2. Create a RHEL 9 based Virtual MAchine
3. Perfom Dynamic discovery of VirtualMachine instance to populate a inventory group called `eap` within the `ocpv` inventory.
4. Configure the Virtual Machine with the following:
    1. JBoss Enterprise Application Platform 7.4.0 (and associated dependencies [OpenJDK])
    2. Systemd service for JBoss EAP
    3. Deploy a sample application and make it available at the `/info` context
5. Create an OpenShift _Service_ and _Route_ exposing the JBoss Server

Once the automation has completed successfully, navigate to the `ansible-middleware-ocpv` within the OpenShift web console.

Within Administrator perspective, expand the _Networking_ navigation pane on the lefthand side and select **Routes**. 

Select the **eap-demo** route which will open the exposed VM and present the JBoss EAP welcome page.

Navigate to the `/info` context to view the deployed application.

Enjoy seeing the power of Ansible Middleware setup, configure and automate the deployment of Red Hat Runtimes within the OpenShift Container Platform!