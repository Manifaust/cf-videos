# GCP Prerequisite

* a service account that can deploy BOSH:
  * owner role
	* APIs enabled:
    * Google Identity and Access Management (IAM)
    * Google Cloud Resource Manager APIs
* another service account which will be used by the k8s nodes:
  * `roles/compute.storageAdmin`
  * `roles/compute.networkAdmin`
  * `roles/compute.securityAdmin`
  * `roles/compute.instanceAdmin`
  * `roles/iam.serviceAccountActor`
* VPC network and a subnet with at least /24 CIDR range
* bastion VM so that the scripts we're going to run will have access to the subnet
* NAT VM so that internal VMs can send requests out into internet
  * a route to the NAT
* internal firewall rule so that VMs inside the subnet can talk to each other
