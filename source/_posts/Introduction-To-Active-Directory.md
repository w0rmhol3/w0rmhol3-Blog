---
title: Introduction to Active Directory
date: 2025-02-20 01:17:29
categories: Sharing
author: w0rmhol3
tags: Active Directory
cover: https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/ActiveDirectory/Introduction-To-Active-Directory/Banner.png?raw=true
---

As I am focusing on studying Active Directory for the past few months, I thought that I can share a bit on what I had learnt. I had went through different resources and currently pursuing my CRTP certification, hence the grind. So, lets look into some Active Directory fundamentals.<!--more-->

## Active Directory Security: Understanding The Structure, Enumeration, and Attacks
Active Directory (AD) is a critical component of Windows network infrastructure, enabling centralised management of users, computers, policies, and resources. However, due to its complexity and default configurations, AD environments are often misconfigured, making them a prime target for attackers.

In this blog, I will provide an introduction to the structure of AD, discuss certain enumeration techniques, highlight common attack vectors, and explore methods for privilege escalation within an AD environment.

## What is Active Directory?
`Active Directory is a directory service within a Windows network that provides authentication and authorisation functions.` It allows administrators to manage users, groups, network devices, file shares, policies, and more from a central location. Key resources managed by AD include:
- Users
- Computers
- Groups
- Network devices
- File shares
- Group policies
- Servers
- Workstations
- Domain trusts

Despite its essential role in enterprise networks, AD is not secure by default. Misconfigurations can lead to significant vulnerabilities that attackers can exploit.

## Active Directory Enumeration
Even without additional privileges, a `basic domain user` can enumerate a substantial portion of an AD environment, gaining insight into its security posture. Some key areas that can be enumerated include:
- Domain Computers – List all machines within the domain.
- Domain Users – Identify all user accounts.
- Domain Groups – Understand group memberships and permissions.
- Default Domain Policies – Review security policies.
- Domain Functional Levels – Determine AD version and compatibility.
- Password Policies – Analyse password complexity and expiration policies.
- Group Policy Objects (GPOs) – Investigate security and access control policies.
- Kerberos Delegation – Identify accounts with unconstrained delegation.
- Domain Trusts – Map relationships between domains.
- Access Control Lists (ACLs) – Examine permissions and privilege assignments.

This enumeration process is crucial in assessing security weaknesses, identifying overly permissive policies, and uncovering potential privilege escalation paths.

## Common Active Directory Attacks
`Misconfigurations and poor security practices` make AD environments `vulnerable` to various attacks. Below are some of the most common AD attack techniques:

- `Kerberoasting` / ASREPRoasting – Extracting service account credentials from Kerberos tickets.
- `NTLM Relaying` – Intercepting authentication attempts and relaying them to gain access.
- `Network Traffic Poisoning` – Exploiting network protocols to capture credentials.
- `Password Spraying` – Trying common passwords across multiple accounts to bypass lockout policies.
- `Kerberos Delegation Abuse` – Exploiting accounts with excessive delegation rights.
- `Domain Trust Abuse` – Leveraging trust relationships to move laterally.
- `Credential Theft` – Extracting credentials from memory or disk.
- `Object Control` – Exploiting weak ACLs to manipulate AD objects.

Understanding these attack techniques is essential for both offensive security professionals and defenders aiming to secure AD environments.

## Active Directory Structure
AD follows a `hierarchical tree structure`, with the largest unit called a Forest, which acts as the security boundary. Within the Forest, there are multiple Domains, each containing Objects such as users, computers, and groups.

`Forest` – The top level, containing one or more domains.
`Domains` – The next level within the forest, holding objects like users and computers.
`Organizational Units (OUs)` – Containers within a domain that store objects and group policies.
`Objects` – The fundamental data units, such as users, computers, and groups.

![Hierachy View (from HTB)](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/ActiveDirectory/Introduction-To-Active-Directory/Hierachy.png?raw=true)
Here's another view of what it looks like:
![File System View](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/ActiveDirectory/Introduction-To-Active-Directory/File-System.png?raw=true)

## Active Directory Trust Relationships
Trust relationships define how authentication flows between different domains and trees within an AD forest. These relationships determine whether users from one domain can access resources in another.

`Types of Trusts:`
- Transitive Trust – Allows trust to extend beyond two domains (e.g., if A trusts B and B trusts C, then A trusts C).
- Non-Transitive Trust – Limits trust to only two domains (A trusts B, but B’s trust with C does not extend to A).
- One-Way Trust – Only one domain trusts the other.
- Two-Way Trust – Both domains trust each other.
- Parent-Child Trust – A two-way transitive trust between a parent and a child domain.
- Tree Root Trust – A two-way transitive trust between trees within a forest.

![Trust Relation](https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/ActiveDirectory/Introduction-To-Active-Directory/Trust-Relation.png?raw=true)

## Foothold Enumeration in AD
Once a foothold is gained within an AD environment, key information should be gathered to identify potential escalation paths:
- Domain functional levels
- Password policies
- Inventory of AD users, computers, and groups
- Domain trust relationships
- Object ACLs
- Group Policy Objects (GPOs)
- Remote access rights

Weak configurations, such as improper use of jump hosts or misconfigured Citrix servers, can provide opportunities for lateral movement and privilege escalation.

## AD Privileges and Rights
Different groups within AD have varying levels of privileges. Some of the most critical groups include:
|Group                      |Importance                                                         |
|---------------------------|-------------------------------------------------------------------|
|Default Administrators     |Superuser group (Domain Admins, Enterprise Admins)                 |
|Server Operators           |Can modify services, access shares, and back up files              |
|Backup Operators           |Can log into DCs locally and act as Domain Admins                  |
|Print Operators            |Can load malicious drivers into DCs                                |
|Hyper-V Administrators     |	Can control virtual domain controllers                          |
|Account Operators          |Can modify non-protected accounts and groups                       |
|Remote Desktop Users       |May facilitate lateral movement through RDP                        |
|Remote Management Users    |Can log into DCs via PowerShell Remoting                           |
|Group Policy Creator Owners|Can create new GPOs but require additional permissions to link them|
|Schema Admins              |Can modify AD schema, allowing backdoor creation                   |
|DNS Admins                 |Can inject malicious DLLs or exploit WPAD configurations           |

## Remote Server Administration Tools (RSAT)
RSAT enables administrators to `remotely manage AD and other Windows Server roles from a workstation.`

`Key RSAT Tools:`
- Active Directory Users and Computers (ADUC)
- Group Policy Management Console (GPMC)
- DNS Management Tools
- DHCP Management Tools
- Hyper-V Management Tools
- Remote Desktop Services Tools
- Windows Server Update Services Tools

`PowerShell Commands for RSAT:`

List all available RSAT tools:
```sh
Get-WindowsCapability -Name RSAT* -Online | Select-Object -Property Name, State
```
Install all RSAT tools:
```sh
Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability –Online
```
Install a specific RSAT tool:
```sh
Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 –Online
```

## The Power of NT AUTHORITY\SYSTEM
The `NT AUTHORITY\SYSTEM account` has the highest level of privilege on a Windows machine, exceeding that of local administrators. It can:

- Run most Windows services
- Impersonate computer accounts on a domain-joined host
- Perform privilege escalation techniques
Gaining SYSTEM-Level Access
- Exploiting remote Windows vulnerabilities (e.g., EternalBlue, BlueKeep)
- Abusing running services in SYSTEM context
- Leveraging SeImpersonate privileges with RottenPotatoNG
- Exploiting local privilege escalation flaws (e.g., Task Scheduler vulnerabilities)
- Using PsExec with the `-s` flag

Examples:

Running a process as SYSTEM:
```sh
psexec.exe -s -i cmd.exe
```
Enumerating domain info as SYSTEM:
```sh
nltest /dclist:domain
```

## Conclusion
Active Directory security is a critical component of an organisation's overall security posture. By understanding its structure, identifying common enumeration techniques, and recognising attack methods, organisations can implement stronger security controls to mitigate risk. Whether you're a penetration tester or a defender, staying ahead of AD threats is essential in today’s cybersecurity landscape.