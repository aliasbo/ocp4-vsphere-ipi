# OpenShift Installer for vSphere IPI

This ansible playbook assists on the installation of OpenShift on vSphere using IPI.

The execution has this workflow:

- Configure the bastion host where the installation is initiated.
- Validate the pre-requisites for a regular IPI installation.
- Generate the `install-config.yaml` file for the installer.
- Run the `openshift-install` program.
- Print the cluster api and web console.

> The playbook is targeted to run in an already provisioned Linux host (called bastion). Where the running user has sudo privileges. And the host has internet connectivity and access to the package repository of the distribution.

## Clone the repo

Clone the repo.

```shell
git clone https://GIT-REPO.git
```

Change to the repo directory

```shell
cd GIT-REPO-DIR
```

## Create a vault file

The file `vault` will store the Pull Secret and the password for vCenter.

Download the pull-secret from the [Pull Secret page](https://cloud.redhat.com/openshift/install/pull-secret) on the Red Hat OpenShift Cluster Manager site and save it to

Move the pull-secret file to your current path.

Create the vault file with the content of the pull_secret.

```shell
echo "vault_pull_secret: $(< pull-secret)" > host_vars/localhost/vault
```

Save the vCenter password to a file called `vcenter_password`.

Add the content of the password file to the vault file.

```shell
echo "vault_vcenter_password: $(< vcenter_password)" >>  host_vars/localhost/vault
```

Create `.vault_pass` to store the vault password.

```shell
echo "vault_password" > .vault_pass

```
Encrypt the file. Ansible will look for `.vault_pass` at the main dir of the repository.

```shell
ansible-vault encrypt host_vars/localhost/vault
```

View the file to confirm the data is correct.

```shell
ansible-vault view host_vars/localhost/vault
```

## Create the inventory file

Create a copy of the reference inventory file into `inventory`.

```shell
cp inventory_reference.yml inventory/openshift_cluster.yml
```

Edit the new file to set the value of the required objects. The commented objects are optional.

```shell
vim inventory/ocp_cluster.yml
```
```yaml
all:
  hosts:
    localhost:
  vars:
##### Name of the cluster
    cluster_name: <ocp4>
##### dns domain for the cluster
    base_domain: <example.com>
##### IP assigned to the DNS record api.<cluster_name>.<base_domain>
    api_ip: <'192.168.1.8'>
##### IP assigned to the DNS record *.<cluster_name>.<base_domain>
    apps_ip: <'192.168.1.9'>
##### IP address of the NTP server
    ntp_ip: <'192.168.1.1'>
##### IP address of the DNS server
    dns_ip: <'192.168.1.1'>
##### vCenter access details
    vcenter_ip: <'192.168.22.10'>
    vcenter_username: <'administrator@vsphere.local'>
    vcenter_datacenter: <Datacenter>
    vcenter_cluster: <Cluster>
    vcenter_datastore: <Datastore>
    vcenter_network: <NetworkName>
```

Run the `main.yaml` playbook using the previous inventory.

```shell
ansible-playbook -i inventory/openshift_cluster.yml main.yml
```
