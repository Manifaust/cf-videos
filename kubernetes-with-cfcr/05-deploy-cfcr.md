## Deploy CFCR

### Set up k8s networking infrastructure

Execute the terraform plan that will set up the networking infrastructure for our kubernetes cluster.

```sh
$ cd ~
$ cp kubo-deployment/docs/user-guide/routing/gcp/kubo-lbs.tf ./

# specify where the terraform state will be created
$ export cfcr_terraform_state=~/cfcr-config/terraform.tfstate
```

Next, you'll need to apply the IaaS terraform script. It takes as input some existing settings such as your network name and project ID. If you used the the terraform plan from the docs to set up the bastion, these properties will already be available to you as environment variables on the machine.

```sh
$ terraform apply \
    -var network=${network} \
    -var projectid=${project_id} \
    -var region=${region} \
    -var prefix=${prefix} \
    -var ip_cidr_range="${subnet_ip_prefix}.0/24" \
    -state=${cfcr_terraform_state}
```

This plan creates a target pool for the k8s master, a load balancer and firewall rule for ingress into the master. You can find them in the GCP cloud console.


### Update `director.yml`

`director.yml` will be reused when we deploy our k8s cluster. First we need to use terraform to recall the target pool name and load balancer address.

```sh
# terraform has created a GCP instances pool to put our master
$ export master_target_pool=$(terraform output -state=${cfcr_terraform_state} kubo_master_target_pool)

# it has also created a LB in front of that pool
$ export kubernetes_master_host=$(terraform output -state=${cfcr_terraform_state} master_lb_ip_address)
```

Then, open up `director.yml` and fill in the related properties.

```yaml
routing_mode: iaas
kubernetes_master_host: <replace with $kubernetes_master_host>
master_target_pool: <replace with $master_target_pool>
```

### Deploy our k8s cluster

Finally, it's time to deploy k8s. We have the handy script `deploy_k8s` which will uses the CFCR BOSH release to configure and launch our cluster. It'll create VMs for our k8s master and workers, set up the CAs and certs correctly, and well as launch all the k8s processes.

```sh
$ kubo-deployment/bin/deploy_k8s ~/cfcr-config my-k8s-cluster
```

The deployment process takes about twenty minutes on GCP. When it's complete, BOSH will begin monitoring VMs, maintaining logs, and restart any crashed components.

If you look at the GCP cloud console, you'll see four new VMs. If you look at the their tags, you'll be able to tell that one of them is a k8s master, and the others are k8s nodes.

### Set Up kubeconfig

To communicate to the cluster, you'll see to set up a kubeconfig. We have script `set_kubeconfig` that does it for you.

```sh
$ kubo-deployment/bin/set_kubeconfig
$ kubo-deployment/bin/set_kubeconfig \
    ~/cfcr-config my-k8s-cluster
$ kubectl get svc --all-namespaces
$ less .kube/config
```

You can now access the k8s cluster from anywhere by exporting the kubeconfig to the machine you want to use. Exit the bastion and use the `gcloud compute scp` command to copy the kubeconfig.

```sh
$ gcloud compute scp k1-bosh-bastion:~/.kube/config ./kubeconfig
$ kubectl --kubeconfig=./kubeconfig cluster-info
$ kubectl --kubeconfig=./kubeconfig get componentstatuses
$ kubectl --kubeconfig=./kubeconfig get nodes
$ kubectl --kubeconfig=./kubeconfig get svc --all-namespaces
```
