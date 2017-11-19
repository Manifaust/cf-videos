# Deploy BOSH

## SSH into bastion

```sh
gcloud compute ssh k1bosh-bastion
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

## Create the encryption key for credhub

```sh
random_key=$(hexdump -n 16 -e '4/4 "%08X" 1 "\n"' /dev/urandom)
# edit the director.yml
# credhub_encryption_key: ${random_key}
```

## Use RBAC

```yaml
authorization_mode: rbac
```

## Start the deployment

```sh
kubo-deployment/bin/deploy_bosh \
    ~/cfcr-config ~/gcp-service-account.key.json
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