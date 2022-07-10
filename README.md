# ocp-dev-cluster
Utility to create OCP cluster on libvirt environment, for development purpose 

# Pre-requisites

- RHEL 8 host (Tested with 8.5)

## Preparation

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

    Go to https://console-openshift-console.apps.ci.l2s4.p1.openshiftapps.com/, click on your name in the top
right, copy the login command, extract the token from the command and
use it to set `CI_TOKEN` environment variable.
    ```bash
    export CI_TOKEN=<extracted_value>
    ```

5. Export PULL_SECRET_PATH Environment Variable (absolute file path)

    Save the secret obtained from [cloud.openshift.com](https://cloud.redhat.com/openshift/install/pull-secret)
and export the file path
    ```bash
    export PULL_SECRET_PATH=<pull_secret_file_path>
    ```

6. Clone ocp-dev-cluster repository and change directory to it
    ```bash
    git clone https://github.com/nirarg/ocp-dev-cluster.git
    ```

7. In case you need, you can change any Environment Variable configured in [dev-cluster](dev-cluster)

### Usage

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

### More Info

https://docs.google.com/document/d/1pxHJ4elXjRVgEZXK5U1kSmMQd21wMcXFO567ORzgCKY/edit?usp=sharing
