# ocp-dev-cluster
Utility to create OCP cluster on libvirt environment, for development purpose

# Machine Requirements

- RHEL 8 host (Tested with 8.5 and 8.6)

# Preparation

## Get your CI Token
Go to https://console-openshift-console.apps.ci.l2s4.p1.openshiftapps.com/, click on your name in the top right and copy the login command, extract the token from the command

## Get your Pull Secret
You can download your Pull Secret from [cloud.openshift.com](https://cloud.redhat.com/openshift/install/pull-secret)

# Pre-requisites Installation
You can either use the provided `Ansible` playbook or run the steps manually

## Using Ansible
1. If you do not have root access to On your target machine enable passwordless sudo for the current user

    Consider creating a separate user for deployments, one without SSH access.

    `echo "$USER  ALL=(ALL) NOPASSWD: ALL" >/etc/sudoers.d/${USER}`


2. Clone ocp-dev-cluster repository and change directory to it
    ```bash
    git clone https://github.com/nirarg/ocp-dev-cluster.git
    cd ocp-dev-cluster/ansible
    ```

3. Prepare your `inventory.yml` file:

    If you are running Ansible from your development machine:
    ```yaml
    all:
      hosts:
        <Host Name>:
          ansible_connection: ssh
          ansible_user: <Username to connect with>
          ansible_ssh_private_key_file: <Location of your Private SSH key>
          ansible_become: <Set to false if connecting as "root", otherwise remove>
          ansible_python_interpreter: /usr/libexec/platform-python
    ```
    If you are running Ansible from the target machine
    ```yaml
    all:
      hosts:
        127.0.0.1:
          ansible_connection: local
          ansible_become: <Set to false if running as "root", otherwise remove>
    ```

4. Prepare your `values.yml` file:
    ```yaml
    ci_token: <Your CI Token>
    rh_username: <Your RedHat Username to subscribe with>
    rh_password: <Your RedHat Password to subscribe with>
    pull_secret_file: <Path to where you saved your Pull Secret file>
    ```

    By default, the playbook will run `dev-cluster all`. If you wish to run a specific command, set `dev_cluster_command` in your `values.yml` file to the desired command. See [here](README.md#usage) for details

    The README uses a plain-text `values.yml` file. Consider using `ansible-vault` as the file includes sensitive information

    4.1. You may optionally set the desired OCP version be setting
    ```yaml
    ocp_version: <Your desired OCP version>
    ```
    See [here](#finding-supported-openshift-versions) for details

    4.2. You may optionally set environment variable you need to be used in the cluster creation

    for example:
    ```yaml
    os_environment:
      - key: NETWORK_TYPE
        value: OVNKubernetes
      - key: SSH_PUB_KEY
        value: "ssh-..."
    ```
    For more environment variables options see [config_example in dev-scripts](https://github.com/openshift-metal3/dev-scripts/blob/master/config_example.sh)

    4.3 The Ansible playbook also supports setting up HAProxy to allow direct access to the cluster from the bare metal machine.

    To setup HAProxy set:
    ```yaml
    configure_haproxy: true
    ```

5. Execute Ansible Playbook

    With Root Access
    ```bash
    ansible-playbook -e @values.yml playbook.yml
    ```

    Without Root Access
    ```bash
    ansible-playbook -e @values.yml --ask-become-pass playbook.yml
    ```

6. You may find the kubeconfig file on the target machine at `~/ansible/ocp-dev-cluster/dev-scripts/ocp/my-cluster/auth/kubeconfig`

## Manual
Considering that this is a new install on a clean OS, the next tasks should be performed prior the installation:

1. Enable passwordless sudo for the current user

    Consider creating a separate user for deployments, one without SSH access.

    `echo "$USER  ALL=(ALL) NOPASSWD: ALL" >/etc/sudoers.d/${USER}`

2. In case of RHEL, invoke `subscription-manager` in order to `register` and `attach` the subscription

3. Install new packages

    ```bash
    sudo dnf upgrade -y
    sudo dnf install -y git make wget jq
    ```

4. Export CI_TOKEN Environment Variable

    Get your CI Token as explained [here](README.md#get-your-ci-token) and
use it to the set `CI_TOKEN` environment variable.
    ```bash
    export CI_TOKEN=<extracted_value>
    ```

5. Export PULL_SECRET_PATH Environment Variable (absolute file path)

    Get your Pull Secret as explained [here](README.md#get-your-pull-secret)
and export the file path
    ```bash
    export PULL_SECRET_PATH=<pull_secret_file_path>
    ```

6. Clone ocp-dev-cluster repository and change directory to it
    ```bash
    git clone https://github.com/nirarg/ocp-dev-cluster.git
    cd ocp-dev-cluster
    ```

7. If you wish to set the OCP version to a specific GA version, you may set the `OCP_VERSION` environment variable
    ```bash
    export OCP_VERSION=<Your desired OCP version>
    ```
    See [here](#finding-supported-openshift-versions) for details

8. In case you need, you can change any Environment Variable configured in `dev-cluster`

## Finding supported OpenShift versions

Look [here](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/) for the release versions

# Usage

```bash
./dev-cluster <cammand>

create-cluster     ==> Create new OCP cluster
install-nfs-class  ==> Install NFS Storage Class on the cluster
install-metallb    ==> Install MetalLB (LoadBalancer) on the cluster
install-cnv        ==> Install CNV (Openshift Virtualization) on the cluster

all                ==> Create New OCP cluster and install NFS StorageClass, MetalLB and CNV on it
clean              ==> Delete the OCP cluster
help               ==> Print usage
```

# Debugging your fork

## Debugging Ansible
To debug your fork, you may set the following in your `values.yaml` file to make the ansible playbook use your branch
```yaml
repo_to_clone: < The URL for your fork >
version_to_clone: < Name of your branch >
```

### More Info

https://docs.google.com/document/d/1pxHJ4elXjRVgEZXK5U1kSmMQd21wMcXFO567ORzgCKY/edit?usp=sharing
