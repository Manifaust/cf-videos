# Deploy CFCR

## Set Up Networking Infrastructure

Execute the terraform plan that will set up the networking infrastructure for our kubernetes cluster.

```sh
$ cd ~
$ cp kubo-deployment/docs/user-guide/routing/gcp/kubo-lbs.tf ./

# specify where the terraform state will be
$ export cfcr_terraform_state=~/cfcr-config/terraform.tfstate

$ terraform apply \
    -var network=${network} \
    -var projectid=${project_id} \
    -var region=${region} \
    -var prefix=${prefix} \
    -var ip_cidr_range="${subnet_ip_prefix}.0/24" \
    -state=${cfcr_terraform_state}

# terraform has created a GCP instances pool to put our master
$ export master_target_pool=$(terraform output -state=${cfcr_terraform_state} kubo_master_target_pool)

# it has also created a LB in front of that pool
$ export kubernetes_master_host=$(terraform output -state=${cfcr_terraform_state} master_lb_ip_address)
```

## Append Configuration

The `director.yml` will be re-used to deploy CFCR as well. We just need to provide some new configuration regarding our pool and LB.

```yaml
routing_mode: iaas
kubernetes_master_host: ${kubernetes_master_host}
master_target_pool: ${master_target_pool}
```

## `deploy_k8s`

Execute the script that deploys the CFCR BOSH release

```sh
kubo-deployment/bin/deploy_k8s ~/cfcr-config my-k8s-cluster
```

## Set Up kubeconfig

```sh
kubo-deployment/bin/set_kubeconfig
kubo-deployment/bin/set_kubeconfig \
    ~/cfcr-config my-k8s-cluster
kubectl get svc --all-namespaces
less .kube/config
```

Exit the bastion.

```sh
gcloud compute scp k1-bosh-bastion:~/.kube/config ./kubeconfig
kubectl --kubeconfig=./kubeconfig cluster-info
kubectl get componentstatuses
kubectl get nodes
kubectl --kubeconfig=./kubeconfig get svc --all-namespaces
```
