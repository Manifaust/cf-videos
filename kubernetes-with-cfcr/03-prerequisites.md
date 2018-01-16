## GCP Prerequisite

For this guide we're going to be using GCP. (However, the official CFCR docs also describe how to get it working in AWS, vSphere, and OpenStack.) Before we get started, here are some prerequestites you'll need from

* A GCP project, in my case this is `cf-sandbox-twong`.
* APIs enabled:
  * Google Identity and Access Management (IAM)
  * Google Cloud Resource Manager APIs
* A service account that can deploy BOSH, it has to have the `Owner` role. In this guide we'll call this service account `k1-bosh@cf-sandbox-twong.iam.gserviceaccount.com`
* A VPC Network. We'll call it `cfcr-net` in this example.

The official docs have a terrform plan that will create the next set of prerequisites for you. However, if you don't want to use terraform, I've listed out the individual elements you need below.

* Another service account which will be used by the k8s nodes. In our case we'll call it `k1-node@cf-sandbox-twong.iam.gserviceaccount.com` and it needs these roles:
  * `roles/compute.storageAdmin`
  * `roles/compute.networkAdmin`
  * `roles/compute.securityAdmin`
  * `roles/compute.instanceAdmin`
  * `roles/iam.serviceAccountActor`
* A subnet under `cfcr-net` with at least /24 CIDR range. In our case we'll call the subnet `k1-us-west1-subnet`.
* A [bastion VM](https://cloud.google.com/solutions/connecting-securely#bastion) so that the scripts we're going to run will have access to the subnet. We'll name this VM `k1-bosh-bastion`.
* A firewall rule so that we'll have SSH access into the bastion.
* A NAT VM so that internal VMs can send requests out to the internet. We'll name this VM `k1-nat-instance-primary`.
* A route to our NAT VM. We'll call this route `k1-nat-primary`. Furthermore, we'll configure it so that all VMs with a tag called `no-ip` will use this route.
* A firewall rule so that VMs inside the subnet can talk to each other. If you use the terraform plan, all VMs with the tag `internal` shares this firewall rule.

With all the prequisites out of the way, we're now ready to install BOSH.