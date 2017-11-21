# How does BOSH work?

[BOSH architecture](https://bosh.io/docs/bosh-components.html)

Like k8s, BOSH is made up of several components. In fact, if you understand the components of k8s, you can often find direct analogues in BOSH.

* Director (like a k8s master) - receives command from the user and creates tasks to be run. It reconciles the current state of the system and the expected state.
* Agent (a process running on each VM) - think of it as kubelet, takes orders from the director, checks with Monit to make sure processes on the VM are alive. Communicates with the health monitor
* Health Monitor
* CPI
* CLI
* BOSH release

BOSH is great for deploying platforms, and that’s why it’s used to operating something as complex as Cloud Foundry and now k8s.


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