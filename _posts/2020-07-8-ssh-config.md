---
title: 'Make life easier with .ssh/config'
date: 2020-07-08
permalink: /posts/2020/07/post-ssh-config/
tags:
  - ssh
  - remote working
  - hpc
  - jump client
---

Often, you aneed to connect to a dozen of HPCs remotely fron a local UNIX environment (your laptop or woekstation) on a daily basis. For this purpose, you struggle to remember all these various usernames, remote addresses and command line options. You can of course create alias in your .bashrc (or .zshrc for macOS) file to make it easier. However, the `~/.ssh/config` file provides a more flexible and elegent way to make your life much easier. 

This file is usually here: `~/.ssh/config`, and you can edit this file following my examples below

## A simple example to start with

By putting the following into your `~/.ssh/config`, you can simply connect to HPC1 by `$ ssh HPC1`.
<pre>
Host HPC1
	HostName HPC1.exaple.net
	Port 2222
	User HPC1_user_ID
</pre>>

## To make global configurations
<pre>
Host *
	ServerAliveInterval 30
	ServerAliveCountMax 3
	# you might need to create the directory ~/.ssh/controlmasters/
	ControlPath ~/.ssh/controlmasters/%r@%h:%p
	ControlMaster auto
	ControlPersist yes
	# allow x forwarding to interact with GUI windows
	ForwardX11 yes
	ForwardAgent yes
	ForwardX11Trusted yes
	#AllowTcpForwarding yes
	#GatewayPorts yes
</pre>
The above will apply to all of your hosts in your `~/.ssh/config` file.


## Jump using ProxyJump
Now, suppose you need to conenct to HPC2. However, you are not allowed to do that from your local computer directly. Instead, you have to first jump to HPC1 and then jump to HPC2 from HPC1. Easy peasy:
<pre>
Host HPC2
	HostName HPC2.example.com
	User HPC2_user_ID
	ProxyJump HPC1
</pre>
After this, you can connect to HPC2 by `$ ssh HPC2` from your local computer. Great right. 

### Jump to remote HPCs that require loading private key to ssh-agent
Let's look at a more complicated scenario. There are cases when you cannot jump to HPC2 via HPC1 strightforwardly. Instead, you need to run a command before after conencted to HPC1 and before connecting to HPC2. 
Here we take the UK HPC [JASMIN](https://help.jasmin.ac.uk/article/187-login) as an example. To log into JASMIN, you first need to be on an academic institutional UNIX environment (let's call it HPC1), from there you need to load the private key into ssh-agent on HPC1 each time before ssh to JASMIN. Hassle? In fact, you can connect to JASMIN from you local computer in just one step: 

- Step 1: Generate your SSH key pair as usual on HPC1 where you need to load your private key itno ssh-agent in order to ssh to HPC2. Note you may need to refer to your remote host instrucitons in terms of how to generate SSH key pair. 
- Step 2: Copy the generated **private key** (normally named `id_rsa*`, NB not your public key which has a name like `id_rsa*.pub`) to your lcoal compute under the `~/.ssh` folder using either scp or rsync. Then change the permission of the copied private key as 400: `$ chmod 400 id_rsa*`
- Step 3: Edit your `~/.ssh/config` file such that you can jump to HPC2 from HPC1
	<pre>
	Host HPC2
		HostName HPC2.example.com
		User HPC2_user_ID
		ProxyJump HPC1
	</pre>

- Step 4: On your local computer, load the copied private key to ssh-agent:
	<pre>
	$ eval $(ssh-agent -s)
	$ ssh-add ~/.ssh/id_rsa_jasmin
	</pre>
	And then, on your local computer, ssh to HPC2 by `ssh HPC2`
	
	You can make this even simpler by creating a tubed alias in `.bashrc` (or `.zshrc` for MAC): 
	`alias Jump2HPC2="eval $(ssh-agent -s) &&  ssh-add ~/.ssh/id_rsa_jasmin && ssh -A HPC2"`. 
	
	Therefore, each time when you do `$ Jump2HPC2`, you are connected to HP2.


