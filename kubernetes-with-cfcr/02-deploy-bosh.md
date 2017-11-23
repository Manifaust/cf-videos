# How does BOSH work?

[BOSH architecture](https://bosh.io/docs/bosh-components.html)

Like Kubernetes, BOSH is made up of several components. In fact, if you understand the components of Kubernetes, you can often find direct analogues in BOSH.

* Director (like a Kubernetes master) - receives command from the user and creates tasks to be run. It reconciles the current state of the system and the expected state.
* Agent (a process running on each VM) - think of it as kubelet, takes orders from the director, checks with Monit to make sure processes on the VM are alive. Communicates with the health monitor
* Health Monitor - monitors the health of VMs
* Cloud Provider Interface (CPI) - talks to a IaaS to procure resources
* CLI
* BOSH release

BOSH was designed to opearate complex systems, that’s why it's relied on to operate Cloud Foundry and now Kubernetes.

## BOSH Release

While BOSH has many similarities with Kubernetes, the way they package software is different. You can think of packaged software in the BOSH world as tarballs called *releases*. Releases contains the libraries, source code, binaries, scripts, and configuration templates needed to deploy a system of software. 

In case the case of the CFCR BOSH release, the packages in the tarball include golang, cni, flanneld, among others. The binaries include api-server, kubeproxy, kubelet, among others. The release also include configuration templates to configure these components. The whole thing is packaged together in a tarball, and the software vendor can then distribute this tarball however they like.

When the user deploys this release, the resulting instance is called a deployment. The way a deployment is created goes like this:

1. taking the release, compiling packages and source code
1. spin up the configured number of VMs
1. start the BOSH agent on each VM
1. install the of the compiled software on the VMs
1. set up the configuration for those software
1. start the software, also start monitoring for crashes

When a new version of a release is published, the user goes through the same deployment process, except now a rolling upgrade is possible.

Deploying a director takes around 30 minutes. Upgrading can be faster depending on how much has changed. But it doesn't involve any downtime for the deployments, meaning the services are not disrupted.


# Deploy BOSH

## SSH into bastion

```sh
gcloud compute ssh k1-bosh-bastion
```

## Download kubo-deployment

```sh
wget https://storage.googleapis.com/kubo-public/kubo-deployment-latest.tgz
tar -xvf kubo-deployment-latest.tgz
```

## Create a configuration for deploying BOSH

```sh
kubo-deployment/bin/generate_env_config ~/ cfcr-config gcp
```

This will create the `cfcr-config` folder . Inside of which is the configuration file `director.yml` which contains the properties for deploying BOSH and later for deploying CFCR.

## Update `director.yml` with machine specific configuration

```yaml
project_id: cf-sandbox-twong
network: cfcr-net
subnetwork: k1-us-west1-subnet
zone: us-west1-a
service_account: k1-node@cf-sandbox-twong.iam.gserviceaccount.com # the service account created in the earlier terraform script. It'll be used by the CPI

internal_ip: 10.0.1.252 # decide the future IP address of your director, must be in your subnet
deployments_network: my-cfcr-deployments-network # the internal name that BOSH will use place deployments. Can be whatever you want
internal_cidr: 10.0.1.0/24 # your subnet's CIDR, or at least a subset of it that you want BOSH to use to deploy machines
internal_gw: 10.0.1.1 # your subnet's gateway
director_name: k1-bosh # decide the future name of your BOSH director, can be user friendly and can be whatever you want
dns_recursor_ip: 10.0.1.1 # DNS IP for resolving non-BOSH hostnames
```

Remind people that if they followed the docs to create the bastion this script is already there in `/usr/bin/update_gcp_env`.

## (Opitonal) Use RBAC

Soon, the default authorization mode will be RBAC. For now, if you want to use RBAC instead of ABAC, edit this portion of the `director.yml`

```yaml
authorization_mode: rbac
```

## Kick off the deployment

```sh
kubo-deployment/bin/deploy_bosh \
    ~/cfcr-config ~/k1-admin-service-account.key.json
```

## Show a little of BOSH

```sh
# make sure we have the right version
bosh-cli -v

bosh envs
# each environment is like a kubeconfig

bosh-cli -e cfcr-config deployments
# you'll get an error saying you're unauthorized

cd cfcr-config
ls
# explain director.yml, state.json, and creds.yml
head creds.yml
# copy admin_password

bosh-cli -e cfcr-config login
bosh-cli -e cfcr-config deployments
bosh-cli -e cfcr-config vms
# we're going to come back and see these populated after we deploy CFCR using BOSH
```