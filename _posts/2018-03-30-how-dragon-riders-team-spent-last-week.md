---
layout: post
section-type: post
title: How Dragon Riders team spent last week
category: tech
tags: [ 'bitcoin', 'komodo', 'iguana', 'containers', 'microservices', 'docker', 'openshift', 'kubernetes', 'asnible', 'git']
---

It is 2 days before the end of the Notary Node elections and we would like to share with you how we spent our time in improving Komodo Platform.
If you haven't read our blogpost about our vision yet, you can read it [here]({{ site.baseurl }}{% post_url 2018-03-20-proposed-notary-node-architecture %}){:target="_blank"}.

There was a lot of work on multiple areas:



## Important patches for Iguana
Emmanux was working hard to create these patches to enable 'scalability' on Iguana.
### 'Listen to all' patch
RPC port of current Iguana version listens on localhost (127.0.0.1) interface only. This address is used to send commands to Iguana. Because of this limitation, commands can only be sent from the same node where Iguana is running. This [patch]( https://github.com/Emmanux/kmdplatform/blob/master/iguana/docker/listen_to_all.patch ){:target="_blank"} solves the problem by enabling connections from all origins.


### 'Talk to url' patch
Our goal is to run assetchains on multiple nodes and, in order to achieve that, this [patch]( https://github.com/Emmanux/kmdplatform/blob/master/iguana/docker/talk_to_url.patch ){:target="_blank"} solves another limitation where Iguana talks only to localhost assetchains.

We need a proper solution, where the url address can be specified in each assetchain config file.

### Monitoring
We're exploring options how to monitor containers in a reliable way.

## Configurable Assetchains configuration files
This is still an ongoing activity which connects directly to previous Iguana issues.
With current source code, Iguana is expecting that assetchains configuration files are accessible at:
<pre>
<code data-trim class="bash">~/.komodo/ASSET_NAME/ASSET_NAME.conf</code></pre>

In our dockerized solution we have a workaround in place, where we mount komodod volume to Iguana container to make assetchain configuration files accessible to Iguana.

In order to scale Notary Nodes, we must be able to run assetchains on different nodes than Iguana instance, and we would like to avoid using shared volume just for sharing configuration files.

This is how our beloved BEER assetchain config file looks like:
<pre>
<code data-trim class="yaml">rpcuser=user2213161329
rpcpassword=passe462292087e033c8dc070d8e0468a58a906e050109ace2c1c6309f719d595b4072
rpcport=8923
server=1
txindex=1
rpcworkqueue=64</code></pre>

rpcuser and rpcpassword values are generated on first start of assetchain. If we want to decouple iguana and komodod without sharing the config volume, iguana must know what is the rpcuser and rpcpassword for each assetchain. We try to solve this dependency by generating configurations for each assetchain, that way we can pass this information to Iguana and create those configurations localy in Iguana container.

Because we already implemented parsing of yaml file [here]( https://github.com/KomodoPlatform/komodotools/blob/master/dragonriders/dokomodo/cli.py#L32 ){:target="_blank"}, we will just add another function which can generate new config files for assetchain.

How it would work
This script then will be bundled into iguana and komodo (assetchain) image and will be set as an entrypoint. Entrypoint in Docker world is the first one executed in the container after starup which can be used for intialization, templatizing config files etc. On each iguana/komodo containers start, entrypoint will be called, which will generate new assetchains configuration files so config files matches.



## Komodotools repository
We are trying form [komodotools]( https://github.com/KomodoPlatform/komodotools ){:target="_blank"} repository which can replace existing 'example' scripts bundled in komodo and Supernet repos. [A-team]( https://github.com/KomodoPlatform/NotaryNodes/tree/master/proposals/a-team ){:target="_blank"} pushed their scripts they use to configure their test notary node. We can build on top of that and improve this so every notary node can benefit from it. We would appreciate if Notary Node candidates can create pull requests with improvements. Intention is also to stop creation of different scripts fullfilling the same function and improve existing ones.

### Future structure of the repository
Our idea is to have these categories in this repository:
- deployment and configuration of the Notary Node OS (Ansible playbooks/roles)
- management tool - different scripts bundled into one piece which allows to 
- monitoring tools - log/metric collection and their visualisation, alerting

I think we should support 2 ways how to deploy NN:
    - baremetal node(s) or vm(s)
    - dockerized installation
Therefore all scripts and tools in this repo should be structured to support that.



### Dokomodo tool
We made dokomodo tool which is still work in progress but suppose to provide 'subcommands' for managing Notary Node platform. This tool takes data from central yaml file, which means code providing functionality and data such as assetchains ports are separated. That means that the addition of a new assetchain won't force you to update your existing scripts. Most of the A-team scripts are written in bash. Some of the scripts are single purpose scripts which must be executed in specific order. This can be improved by using Configuration Management System (CMS).



### Ansible playbooks/roles
We are not going to write/list advantages of CMS, we just name few advantages how Ansible can help us here:
#### Orchestration
Because we all want to scale Komodo Platform, we must consider that the more Notary Node instances, the harder the management will be.
In a multi instance environment it is not sustainable to perform same tasks on different instances, which often leades to human mistakes. By using Ansible inventory file, you can just add a new line with hostname and execute set of playbooks against it.

#### Repeatability / Disaster & Recovery
In case of really bad Notary Node failure, you would need to build a new machine from scratch or perform restore from backup (if you have it :-)), which is not pleasant work to do. 
The recommended way of doing deployments is in a repeatable manner, which in our case would mean that we can rebuild the whole environment. This will give us also the opportunity to constantly test the installation/update procedure.
Our wish is that at some point, we would be able to start new notary nodes by just providing a secret passphrase for the wallet.
