# Prelude

This is the stuff I do to set up the bastion before the start of the video. If you already have a bastion VM you don't need to read this text.

## Create admin service account keys

```sh
$ export prefix=k1
$ export project_id=$(gcloud config get-value project)
$ gcloud iam service-accounts create ${prefix}-admin
$ export service_account_email=${prefix}-admin@${project_id}.iam.gserviceaccount.com
$ gcloud projects add-iam-policy-binding ${project_id} \
    --member serviceAccount:${service_account_email} \
    --role roles/owner
$ gcloud iam service-accounts keys create \
    gcp-service-account.key.json \
    --iam-account ${service_account_email}
```

## Deploy bastion

```sh
$ cp ~/workspace/cf-videos/kubernetes-with-cfcr/kubo-infrastructure.tf ./
$ terraform init
$ export GOOGLE_CREDENTIALS=$(cat gcp-service-account.key.json)
$ export network=cfcr-net
$ export region=us-west1
$ export zone=us-west1-a
$ export subnet_ip_prefix="10.0.1"
$ terraform apply \
    -var service_account_email=${service_account_email} \
    -var projectid=${project_id} \
    -var network=${network} \
    -var region=${region} \
    -var prefix=${prefix} \
    -var zone=${zone} \
    -var subnet_ip_prefix=${subnet_ip_prefix}
```

This creates the following:
* service account `k1-node@cf-sandbox-twong.iam.gserviceaccount.com`
* subnet `k1-us-west1-subnet` with CIDR range `10.0.1.0/24` and gateway `10.0.1.1`
* firewall rule for ssh ingress into the bastion `k1-bosh-bastion`
* firewall rule for communication between bastion and director `k1-intra-subnet-open`
* bastion VM with external IP `k1-bosh-bastion`
* NAT VM with external IP `k1-nat-instance-primary`
* route that allows VMs to use the NAT `k1nat-primary`

## Copy the key into the bastion
```sh
$ gcloud compute scp gcp-service-account.key.json \
    k1bosh-bastion:~/gcp-service-account.key.json
```
