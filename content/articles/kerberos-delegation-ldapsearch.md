---
title: "Enumerating Kerberos Delegations in Active Directory Using ldapsearch"
date: 2026-06-06
description: "Identifying unconstrained, constrained, and resource-based constrained delegation in Active Directory using LDAP queries and PowerShell."
author: "drewbyte"
draft: false
---

*Originally published on [iamdrewbyte.github.io](https://iamdrewbyte.github.io/posts/delegation/)*

**Kerberos Delegation = high-value targets for privilege escalation**

Enumerating Kerberos delegation is a critical step during Active Directory assessments.  
Misconfigured delegation can allow attackers to impersonate users and move laterally across systems.

* * *

## Unconstrained Delegation

```bash
ldapsearch (&(userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samAccountName,userAccountControl
```
 

  * `userAccountControl:...:=524288` → identifies accounts with unconstrained delegation
  * Returns accounts that can impersonate users to any service

## Constrained Delegation

```bash
ldapsearch (&(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo,userAccountControl
```
 

  * `msDS-AllowedToDelegateTo=*` → finds accounts configured for constrained delegation
  * Shows which services the account is allowed to delegate to

## Resource-Based Constrained Delegation (RBCD)

  * Recon with `PowerView`.
  * Find Principals with `WriteProperty` on RBCD Attribute 
```bash
Get-DomainComputer -Server '<TARGET>' | Get-DomainObjectAcl -Server '<TARGET>' | ? { $_.ObjectAceType -eq '3f78c3e5-f79a-46bd-a0b8-9d18116ddc79' -and $_.ActiveDirectoryRights -eq 'WriteProperty' } | select ObjectDN,SecurityIdentifier
```
 
  * Identify the SID 
```bash
Get-DomainObject -LDAPFilter '(objectSid=<SID>)' -Server '<TARGET>'
```
 
  * Alternative (Direct LDAP Check) 
```bash
ldapsearch (&(msDS-AllowedToActOnBehalfOfOtherIdentity=*)) --attributes samAccountName
```
 

> RBCD allows a principal to act on behalf of users to a specific resource

The attribute involved is:

  * `msDS-AllowedToActOnBehalfOfOtherIdentity`

If a `user/group` has `WriteProperty` over this attribute:

  * They can configure delegation
  * Potentially impersonate users to that system
