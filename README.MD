# Inception of things

The goal of the project is to start dabbling with Kubernetes. We will get to familiarize ourselves with the basics of provisioning and CI/CD, and also self-hosting for the bonus part.


## Prerequisite

- Vagrant 2.4.5
- VirtualBox 7.0.18

## Techs and tools

We will use vagrant for provisioning the VMs, Kubernetes for the infrastructure, ArgoCD for the CI/CD (well, obviously more CD than CI), and GitLab for the self-hosting part.
Kubernetes is ... not a basic simple tool. This is why in this project we will only use k3s, not k8s. And that is quite enough for our needs here!

## What the project does

In the 1st part, we simply provision a kubernetes cluster with one server node and one worker node. We want to make sure that we can ssh into each one, and that they belong to the same cluster.

The 2nd part has us work with Ingress and load balancing. We set up 3 web apps in our instance, and which one we access from our browser will depend on the HOST we set up when making a request to the IP address.

The 3rd part is a bit different. Here we won't use Vagrant to create the VMs! This is because we use k3d, which is k3s in Docker (hence the 'd' of "k3d")... It is not listed as a prerequisite here, because we provide a handy script that ensures you have all you need on your machine in order to run this part of the project. The idea here is to use ArgoCD to push a new version of a web app on a github repo... or roll back to previous version if needed. Yes, that's continuous delivery for you.

The bonus is not complex per se : we just want to work with a local gitlab instance instead of github, still using the CI with ArgoCD from part 3. Which means we will install gitlab in our cluster : time to learn about Helm charts !

## 42 evaluation
Let's just start by saying that the hardest part was the bonus, if only because school computer resources are not always where we want them to be : it would take significantly longer to provision/run part 3 at school than our own computers. But even without accounting for this, the configuration for self-hosting was a huge learning step for all of us. 
Anyway, we did complete the full project and its bonus, and earned our 125% grade.