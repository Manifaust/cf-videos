# Intro

*CFCR aims to delight Kubernetes operators*

Kubernetes is great because it lets develops focus on writing and developing software instead of worrying about the infrastructure and platform. 

However, deploying and operating Kubernetes has a few challenges:
* resilience towards IaaS issues
* generate, installing, and managing certificates
* monitoring the health of k8s processes such as proxy and api server
* make the process repeatable (even with different IaaS’s)
* upgrading gracefully or rolling system upgrades

These are the things that CFCR aims to solve. It deals with those tricky issues so operators don’t have to. It accomplishes these goals with the help of BOSH.

## BOSH makes operating distributed systems easy

BOSH is a tool chain for deploying and managing software on VMs. It was designed for distributed, highly available software that has to support multiple cloud providers. It worries about VMs so the operator doesn't have to.

It has these features that are important for k8s:
	* monitoring and health check
	* infrastructure self-healing
	* scaling
	* certificate management and credential management

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
