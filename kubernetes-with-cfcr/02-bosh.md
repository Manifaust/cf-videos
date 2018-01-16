## How does BOSH work?

Like I mentioned before, BOSH is a toolchain that helps people operate complex software. By deploying software with BOSH, your services will have multi-IaaS support, it'll be highly availble, self healing in the face of infrastructure failures, and be easy to scale.

BOSH was designed with these features so it can deploy complex platforms like Cloud Foundry and now Kubernetes. Like Kubernetes, BOSH is made up of several components. In fact, if you understand the components of Kubernetes, you can often find direct analogues in BOSH.

[architecture diagram]

Here're the key [components of BOSH](https://bosh.io/docs/bosh-components.html):

* Director (similiar the Kubernetes API server) - receives commands from the user and creates tasks to be run. It reconciles the current state of the system and the expected state.
* VMs - where BOSH will install software.
* Agent (a process running on each VM) - think of it like a kubelet, agents take orders from the director, checks with Monit to make sure processes on the VM are alive, and communicates with the health monitor.
* Health Monitor - monitors the health of VMs and notifies the director if a VM dies
* Cloud Provider Interface (CPI) - Similar to Kubernetes cloud providers. This is a Cloud Foundry developers have already implemented this interface for all the popular IaaS (including AWS, GCP, vSphere, and OpenStack) and it allows BOSH to talks to each infrastructure  provider to procure resources.
* CLI
* BOSH release

### BOSH Release

While BOSH has many similarities with Kubernetes, the way they package software is different. You can think of packaged software in the BOSH world as tarballs called *releases*. Releases contain the libraries, source code, binaries, scripts, and configuration templates needed to deploy a system of software. 

In case the case of the CFCR BOSH release, the packages in the tarball include Golang, CNI, flanneld, among others. The binaries include api-server, kubeproxy, kubelet, among others. The release also include configuration templates to configure these components. The whole thing is packaged together into a tarball, and the software vendor can then distribute this tarball however they like.

When the user deploys this release, the resulting instance is called a deployment. The way a deployment is created goes like this:

1. Extract the contents of a release, compiling packages and source code
1. Spin up the configured number of VMs
1. Start the BOSH agent on each VM
1. Snstall the of the compiled software on the VMs
1. Set up the configuration for those software
1. Start the software, also start monitoring for crashes

When a new version of a release is published, the user goes through the same deployment process, except now a rolling upgrade is possible.

Deploying a director takes around 30 minutes. Ideally, the user rarely has to upgrade BOSH itself, but an upgrade can be faster than an initial deployment depending on how much has changed. The good thing is it doesn't involve any downtime for the deployments, meaning services are not disrupted.
