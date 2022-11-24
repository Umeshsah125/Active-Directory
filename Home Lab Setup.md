### Installation
1. Install Virtual Box/Vmware WorkStation
2. Download Windows Server 2022 ISO Evaluation Edition (Domain Controller)
3. Download Windows 11 ISO Evaluation Edition (Workstation/Client)
4. Setup both workstation and client on the Virtual Box

### Installing the Domain Controller inside Windows Server 2022
1. Use `sconfig` to:
	- Change the hostname
	- Change the ip address to static
	- Change the DNS Server to our own IP address

2. Install the Active Directory Windows Feature
```shell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```