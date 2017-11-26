# Intro

> *CFCR aims to delight Kubernetes operators*

My name is Tony Wong and I'm an engineer on the Cloud Foundry Container Runtime team, also known as CFCR.

This is a walkthrough of how to install CFCR and deploy Kubernetes with it. CFCR used to be called Kubo, so you'll see lots of references to Kubo in this video. I'm sorry about that, we're fixing that.

The goal of CFCR is to simplify deploying and maintaining a Kubernetes cluster. It was ease of the day-to-day burdens of operators, with features like:

* self-healing against VMs failures or retirement
* generating, installing, and managing certificates
* monitoring the health of Kubernetes processes such as proxy and API server
* support different infrastructure providers so you're not locked down (right now we support GCP, AWS, vSphere, and OpenStack)
* make the deployment, and scaling up process more easily repeatable and easier to automate
* support rolling system upgrades

That's a long list of stuff that operators have to worry about and CFCR is our solution that problem.

How does it do that? Well it accomplishes many of these goals with the help of BOSH.
