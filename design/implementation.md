% Bootstrapping a Secure Infrastructure
% Oliver Mussell
% 2016-2017

<!---

- Produced in Markdown with Vim, converted to HTML by Pandoc.
- Graphics created with DOT
- Hosted on Github Pages

-->

- [Overview](/homelab/design/overview.html)
- [Design](/homelab/design/design.html)
- [Implementation](/homelab/design/implementation.html)


SSH Bootstrapping Problem
---

As part of the provisioning process, the control machine and the new servers it creates must be able to communicate securely with one another, in order to exchange information and configuration data. This must be done without human intervention, so no passwords or interactive sessions. This must also be accomplished as part of the initial bootstrapping, where no other infrastructure yet exists (so no DNS, authentication etc.).

This can be accomplished with SSH, however, the authentication and trust setup of the initial connection and subsequent sessions requires some thought.

We want the control machine, and any other derivative machines to generate their keying material independently. Private keys / passwords should never be sent over the network. 

--- SSH keys ---

As part of the image build process, the public keys of the control machine are included in the image

- Host key in a hashed known_hosts file

	- # Find out the ip of the control machine, then use ssh-keyscan or ssh-keygen to generate the hashed host key.
	- ssh-keygen -F "control_machine_ip" -H | grep -v ^#	
	- # When using NanoBSD:
	- mkdir -p {NANO_WORLDDIR}/root/.ssh
	- output > {NANO_WORLDDIR}/root/.ssh/known_hosts

	- Alternative, pre-compute the host key (since a host key is just a normal private-public key pair, with no password. Also they can be shared and created on other machines.) and add the pre-computed host key to the image. The SSHFP record is generated and added to DNS by the control machine. Then when the control machine connects it can verify using DNS. The problem with this is that it only works if the DNS infrastructure is set up. How are the DNS servers set up in the first place? It also means that the private host key is included in the image build, which will be transmitted over the network to build the server, which is unacceptable.

	- There could be two processes: one for initial bootstrapping and a send once the basic infrastructure is created.


- User keys in an authorized_keys file

	- # The public key needs to go into either root's authorized_keys, or a new user.
	- Allowing login as root opens another security hole. PermitRootLogin needs to be set to yes in sshd_config. May not be that bad since it won't yet have any configuration data. 
	- Adding a new user is also difficult. How do you add a new user when the server is powered off, and you dont have any kind of shell access? You can modify files in the filesystem, but once mounted they are set to read only. This problem only affects the initial setup, because after the basic infrastructure is set up, LDAP can be used for user authentication. 

Once the server has been built and boots, it can generate its host and user keys.

The control machine then connects to the server using its public key to authenticate, and pulls the servers host and user public keys.

	- The server can rotate its host / user keys if they were pre-computed during the image build.

The control machine then adds the SSHFP records to DNS, and the user keys to LDAP 

This means that:

- private keys are not transmitted over the network.
- the control machine can connect securely to the server
- works in the absence of a SSH CA or DNS for validating via a third party

StrictHostKeyChecking vs VerifyHostKeyDNS Problem:
---

Do the StrictHostKeyChecking and VerifyHostKeyDNS options in ssh_config work together?

- StrictHostKeyChecking - If set to yes, ssh will never automatically add host keys to the known_hosts file and refuses to connect to hosts whose host key has changed(This is the preferred option). The host keys of known hosts will be verified automatically in all cases.
- VerifyHostKeyDNS - Specifies whether to verify the remote key using DNS and SSHFP resource records. If set to yes, the client implicitly trusts keys that match a secure fingerprint from DNS. Insecure fingerprints will be handled as if this option was set to ask. If this option is set to ask, information on fingerprint match will be displayed, but the user will still need to confirm new host keys according to the StrictHostKeyChecking option.

So if the fingerprint is presented from insecure DNS (not DNSSEC validated), or if the SSHFP record does not exist, does it prompt the user? We don't want this to happen since these SSH connections are happening autonomously.

Also need to check what happens if both options are set to yes.

If a host key is verified through DNS, is it still added to known_hosts? 





Implementation
===



Provisioning
===

IPv6
---

### Address Autoconfiguration (SLAAC) ###

### SEND ###

### SEND SAVI ###

### DHCPv6 ###

### IPsec ###

### KINK ###

OS
---

### FreeBSD ###

### NanoBSD ###

### Jails ###

### ZFS ###

### Other Operating Systems/Containers/Filesystems ###

### Host Install Tools ###

### Ad-Hoc Change Tools ###

Configuration Management
===

DNS
---

### DNSSEC ###

### DANE ###

### DANE for Email Security ###

### DNSCrypt ###

### DNSCurve ###

LDAP
---

### LDAPS ###

### S/MIME or PGP ###

Kerberos
---


NTP
---

### NTPsec ###


App Deployment
===


NFS
---

### KerberizedNFSv4

Application Servers
---

### NGINX ###


Security and Compliance
===

Security and Crypto
---

### TLS ###

### SSH ###

### HSM ###
### Passwords ###
### TCP Wrapper ###
### IDS ###
### Firewalls ###

Configuration Management Tools
---

Authorisation / Access Control Lists
---

Role-Based Access Control / Shared Administration (sudo)
---

Domain Naming Service
---

Directory Service (LDAP)
---

Time Service
---

Logging / Auditing
---

RPC / Admin service
---

Orchestration
===

Specific Operational Requirements
---

### Configuration ### 

### Startup and shutdown ### 

### Queue draining ###

### Software upgrades ###

### Backups and restores ###

### Redundancy ###

### Replicated databases ###

### Hot swaps ###

### Access controls and rate limits ###

### Monitoring ###

### Auditing ###

Unspecific Operational Requirements
---

### Assigning IPv6 Addresses to Clients ### 

### Static or Dynamic IPv6 Addresses (DHCPv6 or SLAAC) ###

### IPv6 Security ###

### Hostname Conventions ###

### Choosing an Operating System ###

### Choosing a Configuration Management Tool ###

### Scheduling with cron ###

Scaling
===

User Access
===