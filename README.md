# Active-Directory
### I have created this repos to have a better understanding about Active directory and its attack
#### Most part of theory is written from http://www.tryhackme.com website.

What is Active Directory?
-------------------------
-> Active directory is the directory service for windows domain networks. It is used by today's top companies and is a vital skill to comprehend when attacking windows. It is a collection of machines and servers connected inside of domains, that are collective part of a bigger forest of domains, that make up
the active directory network.

Why Active directory is used?
-----------------------------
-> The majority of large companies use Active Directory because it allows for the control and monitoring of their user's computer's through a single  Domain Controller. It allows a single user to sign in any computer on the active directory network and have access to his or her stored files and folders in the server, as well as the local storage on that machine. This allows for any user in the company to use any machine that the company owns,without having to setup multiple users on a machine. 

Major components of Active Directory:
------------------------------------
- Domain Controllers
- Forests, Trees, Domains
- Users + Groups
- Trusts
- Policies
- Domain Services

*Physical Active Directory is the servers and machines on-premise,these can be anything from domain controllers and storage servers to domain user machines; everything needed for an Active Directory environment besides the software.

Domain Controllers:
-------------------
A domain controller is a windows server that has Active  Directory Domain Services(AD DS) installed and has been promoted to a domain controller in the forest. Domain controllers are the center of Active Directory -- they control the rest of the domain.Like-
- holds the AD DS data store
- handles authentication and authorization services
- replicate updates from other domain controllers in the forest
- allows admin access to manage domain resources

AD DS Data Store:
-----------------
- The Active Directory Data store holds the databases and processes needed to store and manage directory information such as users, groups and services.
*Contains the NTDS.dit - a database that contains all of the information of an Active directory domoain controller as well as password hashes for domain users
*Stored by default in %SystemRoot%\NTDS
*accessible only by the domain controller


The Forest
-----------
- The forest is what defines everything; it is the container that holds all of the other bits and pieces of the network together - without the forest all of the other trees and domains would not be able to interact. The one thing to note when thinking of the forest is to not think of it too literally - it is a physical thing just as much as it is figurative thing. When we say "forest" , it is only way of describing the connection created between these trees and domains by the network. Forest is a collection of one or more domains trees inside of an Active Directory network.
Some parts of forest are:

*Trees - A hierarchy of domains in Active Directory Domain Services
*Domains - Used to group and manage objects
*Organizational Units(OUs) - Containers for groups,computers,users,printers and other OUs
*Trusts - Allow users to access resources in other domains
*Objects - users,groups,printers,computers, shares
*Domain Services - DNS Server,LLMNR,IPV6
*Domain Schema - Rules for object creation

Users+Groups
-------------
- Users and groups that are inside of an Active Directory are upto you; when you create a domain controller it comes with default groups and two default users: Administrator and Guest. It is upto you to create a new users and create new groups to add users to.

Users Overview
==============

Users are the core to Active Directory;without users why have active directory in the first place.There are four types of users:
*Domain Admins : They control the domains and only ones with access to the domain controller

*Service Accounts(Can be domain admin) : They are used for service maintainance like SQL service account

*Local Administrator : These users can make changes to local machines as an administrator and may be even able to control other normal users, but they cannot access the domain controller

*Domain Users  : Normal users which may have local admin privilege in some cases

Groups Overview
===============
Groups make it easier to give permissions to users and objects by organizing them into groups with specified permissions. There are mainly two types of AD groups:
*Security Groups : These groups are used to specify permissions for a large number of users
*Distribution Groups : These groups are used to specify email distribution lists. As an attacker these groups are less beneficial but can still be beneficial in enumeration.

Default Security Groups
=======================

    Domain Controllers - All domain controllers in the domain
    Domain Guests - All domain guests
    Domain Users - All domain users
    Domain Computers - All workstations and servers joined to the domain
    Domain Admins - Designated administrators of the domain
    Enterprise Admins - Designated administrators of the enterprise
    Schema Admins - Designated administrators of the schema
    DNS Admins - DNS Administrators Group
    DNS Update Proxy - DNS clients who are permitted to perform dynamic updates on behalf of some other clients (such as DHCP servers).
    Allowed RODC Password Replication Group - Members in this group can have their passwords replicated to all read-only domain controllers in the domain
    Group Policy Creator Owners - Members in this group can modify group policy for the domain
    Denied RODC Password Replication Group - Members in this group cannot have their passwords replicated to any read-only domain controllers in the domain
    Protected Users - Members of this group are afforded additional protections against authentication security threats. See http://go.microsoft.com/fwlink/?LinkId=298939 for more information.
    Cert Publishers - Members of this group are permitted to publish certificates to the directory
    Read-Only Domain Controllers - Members of this group are Read-Only Domain Controllers in the domain
    Enterprise Read-Only Domain Controllers - Members of this group are Read-Only Domain Controllers in the enterprise
    Key Admins - Members of this group can perform administrative actions on key objects within the domain.
    Enterprise Key Admins - Members of this group can perform administrative actions on key objects within the forest.
    Cloneable Domain Controllers - Members of this group that are domain controllers may be cloned.
    RAS and IAS Servers - Servers in this group can access remote access properties of users
    
    
Trusts + Policies
------------------
Trusts are a mechanism in place for users in the network to gain access to other resource in the domain.
Policies are a very big part of Active Directory, they dictagte how the server operates and what rules it will and will not follow

Active Directory Domain Services + Authentication
-------------------------------------------------
- The Active Directory domain services are the core functions of an Active Directory network, they allow for management of the domain, security certificates, LDAPs and much more. They are services that the domain controller provides to the rest of the domain or tree. There is a wide ranges of various services that can be added to a domain controller.
*LDAP - Lightweight Directory Access Protocol; provides communication between applications and directory services
*Certificate services
*DNS,LLMNR,NBT-NS

Domain Authentication Overview
===============================
- The most important part of  Active Directory - as well as most vulnerable part of AD - is the authentication protocols set in place. There are mainly two types of Authentication in AD:
i. Kerberos
- The default authentication service for AD uses ticket-granting tickets and service tickets to authenticate users and give users access to other resources across the domain.

ii. NTLM
- A default windows authentication protocol which uses an encrypted challenge/response protocol
