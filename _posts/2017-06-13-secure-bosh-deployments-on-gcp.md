---
layout: post
title: A more secure approach to BOSH deployments on Google Cloud Platform 
comments: true
type: post
published: true
status: publish
categories:
- Cloud
- Security
- Process
---

*This is a work in progress and may be subject to periodic updates*

BOSH is an OSS tool for deployment, management and monitoring of distributed systems. 
It can work with most Infrastructure-As-A-Service (IASS) providers, I'm going to focus on Google Cloud Platform in this post.

###  *"Manually"* create a bastion instance.
*I say manually here, as this is one of the few steps where we'll use the GCP interface to create an image*

1. Create your cloud project on [https://console.cloud.google.com](https://console.cloud.google.com)
2. Go to the Compute Engine section and create a pretty minimal image, e.g. `g1-small (1 vCPU, 1.7 GB memory)` using whatever linux flavour you wish. Provide it with an external IP address. **THIS WILL BE THE ONLY PUBLIC IP YOU EXPOSE**
3. Go to the Network section and configure the firewall rules to allow port 22 for the instance you just created. Set the IP source whitelisting to be the ip that you plan to manage BOSH from
4. [Generate an ssh key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#platform-linux) for your jumpbox locally, and paste the public output into the ssh field of the VM instance you just created 

### **Create a SOCKS5 proxy**
1. I find the easiest way to set this is up is via a `~/.ssh/config` entry, like this:
	```
	Host my-bastion
  	Hostname <external-ip-address-of-bastion>
  	IdentityFile ~/.ssh/<private-key-generated-above>
  	ForwardAgent yes
  	User <the-username-we-specified-when-generating-the-keypair>
	```
We'll come to why the ForwardAgent is to be used shortly.    
BOSH docs for this step can be found [here](https://bosh.io/docs/cli-tunnel.html)
2. To launch the SOCKS proxy `ssh -D 5000 -fqCN my-bastion`
3. Double check it is launched via `ps -ef | grep ssh` to avoid future frustrations

### **BOSH create-env**
1. Let BOSH know about your newly configured SOCKS proxy via:     
`export BOSH_ALL_PROXY=socks5://localhost:5000`
2. Create a service account on GCP for BOSH giving it the IAM roles:
	- Compute Instance Admin v1
	- Service Account
3. Download the json keys for this account
4. [Optional] If you will need to ssh into the BOSH director at some point in the future, you'll need to add the steps in the next section to your create-env config
5. The following steps can then be followed [https://bosh.io/docs/init-google.html](https://bosh.io/docs/init-google.html)

### **Connecting to the BOSH director via ssh agent forwarding**

Ideally we don't want to leave any credentials on our bastion to limit lateral traversal in the instance of the bastion being compromised. 

1. By including the `-o jumpbox-user.yml` in your `create-env` request BOSH will generate an ssh key pair with the username "jumpbox" (this can be modified in the yml) and store these details in your creds.yml file
2. You can get this data into a file using the BOSH interpolate command :
	```	
	$ bosh int bosh int creds.yml --path /jumpbox_ssh/private_key > director.key
	$ chmod 600 director.key
	```
3. You can now employ ssh agent forwarding thanks to the ForwardAgent setting and some new features in OpenSSH-7      
	`ssh -J my-bastion jumpbox@10.x.x.6`      

This will connect you directly to the director using your bastion key for the first hop and the director.key for the 2nd hop, without persisting that key material to disk on the bastion.  
	
### **Extra bastion hardening**
*This could obviously be exhaustive and I may add notes on [fwknop](http://www.cipherdyne.org/fwknop/) later, but for now just a basic tweak*

1. ssh into the bastion, and tweak the `/etc/ssh/sshd_config` to something like this
```
Port <some-high-port>
PermitRootLogin no
PubkeyAuthentication yes
IgnoreRhosts yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
X11Forwarding no
```

2. Restart the ssh daemon
3. Update the GCP firewall rule to correspond to the high port you just set
4. Update your my-bastion Host entry in ~/.ssh/config to contain    
`Port <some-high-port>`

### **Conclusion**

We just setup a full BOSH environment with the following security benefits:
- Reduced attack surface via the single entry point (ssh bastion)
- Leveraged the "freebie" IP whitelisting to make it more challenging for an attacker to hit that bastion
- No key material on the jumpbox/bastion
- Used the very minimum IAM role that BOSH can operate under to reduce what that account could do if compromised
