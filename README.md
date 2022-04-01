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

Clone this repo.

```shell
git clone https://github.com/USER/REPO.git
```

Change to the repo directory

```shell
cd REPO-DIR
```

## Add sensitive data

The file `vault` will store the password for the vCenter.

Create the `files` directory.

```shell
mkdir files
```

Download the pull-secret from the [Pull Secret page](https://cloud.redhat.com/openshift/install/pull-secret) on the Red Hat OpenShift Cluster Manager site and save it to

Move the pull-secret file to `files`.

```shell
mv pull-secret files/
```

> In case of an __installation from a local registry mirror__. Add the merged pullsecret instead, and copy the certificate of the registry.
> ```shell
> cp ~/merged-pullsecret.json files/
> cp ~/registry.crt files/
> ```

Add the vCenter password to the vault file.

```shell
echo "vault_vcenter_password: SecretPassword" >  host_vars/localhost/vault
```

Create `.vault_pass` to store the password of your choice.

```shell
echo "secret_password" > .vault_pass
```

Encrypt the vault and pull-secret files. Ansible will look for `.vault_pass` at the main dir of the repository.

```shell
ansible-vault encrypt host_vars/localhost/vault
ansible-vault encrypt files/pull-secret
```

View the file to confirm the data is correct.

```shell
ansible-vault view host_vars/localhost/vault

ansible-vault view files/pull-secret
```

## Create the inventory file

Create a copy of the reference inventory file within the `inventory` directory.

```shell
cp inventory/reference.yml inventory/<cluster_name>.yml
```

Edit the new file to set the value of the required objects. The commented objects are optional.

```shell
vim inventory/<cluster_name>.yml
```

Install the required collections using Ansible Galaxy.

```shell
ansible-galaxy collection install -r requirements.yml
```

Run the `main.yaml` playbook using the previous inventory.

```shell
ansible-playbook -i inventory/openshift_cluster.yml main.yml
```

