---
layout: post
section-type: post
title: Proposed Notary Node architecture by Dragon Riders team
category: tech
tags: [ 'bitcoin', 'komodo', 'iguana', 'containers', 'microservices', 'docker', 'openshift', 'kubernetes' ]
---
Komodo elections are in progress and probably every holder is reading Notary Node candidate <a href="https://github.com/KomodoPlatform/vote2018" target="_blank">proposals</a>. Many proposals offer some kind of revenue share which they handout when they get enough votes and win. We at DragonRiders team would like to offer something different to the Komodo community. We are two IT professionals – <a href="https://patchkez.github.io" target="_blank">patchkez</a>  and <a href="https://steemit.com/bitcoin/@emmanux/proposal-for-the-notary-nodes-elections-2018" target="_blank">emmanux</a>  – who want to invest in Komodo Platform by solving every existing scaling issue and adopting innovative technologies and practices from DevOps culture. We already have a concept of how the future architecture of Notary nodes might look like. Here we try to share a few words about current Notary Nodes architecture, possible future bottlenecks and how to solve these problems.


## Overview of current architecture
Current Notary Nodes architecture consists of 64 physical servers located in different regions. Each server runs 3 main components – <a href="https://bitcoin.org/en/bitcoin-core/" target="_blank">Bitcoin Core</a>, <a href="https://github.com/jl777/komodo" target="_blank">Komodod</a> and <a href="https://github.com/jl777/SuperNET" target="_blank">Iguana</a> services. Besides these processes, many assetchains run in parallel, with their own blockchains.  The current production setup allows communication between these services on a single machine, via "port numbers".
Because of all the communications is happening over API (Application Programming Interface), we saw the opportunity to extend the existing functionality to communication between multiple nodes.


## Problem with scaling Komodo Platform
Because of current recommended production setup, Notary Node software can run on single server only. With the increasing number of assetchains, we will come to point where you would need a supercomputer to handle a large number of assetchains.

## Proposed architecture
In modern computing world, in order to allow a platform to scale, it is mandatory that each service run as an independent process and talk to other services via API. Such seggregated processes are often called <a href="https://en.wikipedia.org/wiki/Microservices" target="_blank">microservices</a>.
With such seggregation, it is possible to place microservices on different nodes/servers where communication with other services is done through API via a "shared cluster network". Bitcoin, Komodo and Iguana services already talk to each other via API, we only need to extend their capability so they can run on multiple nodes.
Here is a simple architecture diagram on how scalable Komodo Platform running on top of Kubernetes/Openshift cluster might look like:


![My helpful screenshot]({{ "/img/new_komodo_platform4.png" | absolute_url }})


In order to interconnect, deploy and manage microservices (containers) we need to choose production ready platform. Kubernetes/Openshift to the rescue!

## What is Kubernetes and Openshift
<a href="https://kubernetes.io" target="_blank">Kubernetes</a> is open source platform developed by Google which allows automated deployments, container management and platform scalability. In simple words, this platform allows you to run properly designed applications on multiple nodes/servers. <a href="https://www.openshift.org" target="_blank">Openshift</a> is a opensource container application platform built around Docker containers and Kubernetes cluster management. Openshift project provides additional services we might use while building scalable Komodo Platform.

## Advantages and benefits
Here are some benefits of our proposed solution:

 - Ability to build/run the image in test environment and deploy the same image to production
 - Every image can be versioned (tagged)
 - Rolling back to previous version is also possible in production environment
 - High Availability – containers can be restarted on different a node in the cluster
 - Scalability – new cluster nodes can be added on the fly

## Testnet participation
Because we want to test our nodes and configuration, and want to prove that we will be capable Notary Node operators, we are currently participating in a new Notary Node Testnet. Both of our nodes run Notary Node software in Docker containers. 
