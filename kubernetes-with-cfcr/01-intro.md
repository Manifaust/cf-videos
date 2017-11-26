# Intro

> *CFCR aims to delight Kubernetes operators*

My name is Tony Wong and I'm an engineer on the Cloud Foundry Container Runtime team, also known as CFCR.

CFCR used to be called Kubo, so you'll see lots of references to Kubo in this video. I'm sorry about that, we're slowly converting the names of everything.

The goal of CFCR is to simplify deploying and maintaining a Kubernetes cluster. It was designed to lift the day-to-day burden of operators, with features like:

* providing resilience against VMs failures from your IaaS
* generate, installing, and managing certificates
* monitoring the health of Kubernetes processes such as proxy and API server
* support different infrastructure providers so you're not locked down
* make the deployment process easily repeatable
* support rolling system upgrades

How does it do that? Well it accomplishes many of these goals with the help of BOSH.
