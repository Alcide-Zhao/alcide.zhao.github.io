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

### A simple example to start with
```
Host HPC1
	HostName HPC1.exaple.net
	Port 2222
	User HPC1_user_ID
```
if you put this ito your `~/.ssh/config`, you can simply connect to HPC1 by `$ ssh HPC1`.

### To make global configurations
```
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
```
The above will apply to all of your hosts in your `~/.ssh/config` file.


### Jump using Proxy Jump
Now, suppose you need to conenct to HPC2. However, you are not allowed to do that from your local computer directly. Instead, you have to first jump to HPC1 and then jump to HPC2 from HPC1. Easy peasy:
```
Host HPC2
	HostName HPC2.example.com
    User HPC2_user_ID
    ProxyJump HPC1
```
After this, you can connect to HPC2 by '$ ssh HPC2' from your local computer. Great right. 

### Jump to remote HPCs that require loading private key to ssh-agent
Let's lok at a more complicated scenario. Let's suppose you that you cannot jump to HPC2 via HPC1 strightforwardly. Instead, you need to run a command before doing so. Here we take the UK HPC [JASMIN](https://help.jasmin.ac.uk/article/187-login) as an example. 
To log into JASMIN, you first need to be on an academic institutional UNIX environment (HPC1) and then you need to load your private key into ssh-agent on HPC1 each time before ssh to JASMIN. Complicated? well, the following solve this issue. 

- Step 1:
	Generate your SSH key pair as usual on HPC1 where you need to load your private key itno ssh-agent in order to ssh to HPC2. Note different HPCs may have different routines though. 
- Step 2:
	Copy the generated **private key** (normally named 'id_rsa*', NB not your public key which has a nema like 'id_rsa*.pub') to your lcoal compute under the '~/.ssh' folder using either scp or rsync. Then change the permission of the copied private key as 400: `$ chmod 400 id_rsa*`
- Step 3: 
	Edit your .ssh/config file such that you can jump to HPC2
	```
	Host HPC2
		HostName HPC2.example.com
	    User HPC2_user_ID
	    ProxyJump HPC1
	```
- Step 4
	On your local computer, now load the copied private key to ssh-agent
	```
	$ eval $(ssh-agent -s)
	$ ssh-add ~/.ssh/id_rsa_jasmin
	```
	Then, on your local computer, ssh to HPC2 by executing `ssh HPC2`

	You can make this even simpler by creating a tubed alias in .bashrc (or .zshrc for MAC): `alias Jump2HPC2='eval $(ssh-agent -s) &&  ssh-add ~/.ssh/id_rsa_jasmin && ssh -A HPC2'. Then, when you do `$ Jump2HPC2`, you are connected to HP2.`



