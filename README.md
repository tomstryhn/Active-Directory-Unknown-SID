# Active Directory Unknown SIDs

Unknown SID, Orphaned SID or Unresolvable SID, all three terms cover the same issue, an issue that many AD Administrators, at some point have encountered and/or are battling with, not to mention the hassle to get them all cleared out. I will with along with this Guide supply some code and scripts that might help with this.

# Table of Contents

  - [What is an Unknown/Orphaned/Unresolvable SID?](#what-is-an-unknownorphanedunresolvable-sid)
    - [Basic SID 101](#basic-sid-101)
    - [How do they become Unknown/Orphaned/Unresolvable?](#how-do-they-become-unknownorphanedunresolvable)
  - [Risk(s)](#risks)
  - [Mitigation](#mitigation)
    - [Documented processes on deleting objects in Active Directory](#documented-processes-on-deleting-objects-in-active-directory)
    - [Remember to remove the SIDHistory from your Security Principals](#remember-to-remove-the-sidhistory-from-your-security-principals)
    - [Audit your ACL's on a regular basis](#audit-your-acls-on-a-regular-basis)
  - [Toolbox](#toolbox)
    - [Basic PowerShell commands](#basic-powershell-commands)
    - [Scripts](#scripts)
  - [Links](#links)
      - [<a href="https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/78eb9013-1c3a-4970-ad1f-2b1dad588a25">[MS-DTYP]: SID | Microsoft Docs</a>](#ms-dtyp-sid--microsoft-docs)
      - [ADSecurity](#adsecurity)
      - [AD ACL Scanner](#ad-acl-scanner)

## What is an Unknown/Orphaned/Unresolvable SID?

To understand what it is, you need to have a little insight, and first need to know what a SID is.

### Basic SID 101

SID - or the Security Identifier, is in short a unique identifier used in your Active Directory, among other places. Below is displayed the output when you get the SID from a ADUser, using PowerShell:

```PowerShell

PS C:\> (Get-ADUser Administrator).SID

BinaryLength AccountDomainSid                         Value                                       
------------ ----------------                         -----                                       
          28 S-1-5-21-1234567890-123456789-1234567890 S-1-5-21-1234567890-123456789-1234567890-500

```
> This is a sample SID for the domain administrator of our fictive AD. (Tip: The builtin Administrator always ends with '-500')

A SID uniquely identifies Security Principals, primarily *Computers*, *Groups* and *Users*, there are a few other object-types in your Active Directory that actually have objectsid's but for now we will focus on these three primary object types, since they are also the primary reason for our troubles. As seen on the PowerShell example above, the SID consists of two primary parts, the AccountDomainSid, which identifies the domain to which the object belongs, and the last part, which you could call the serial number for the Security Principal, which is unique in the domain. There are a few more technical details in regard to the SID, but if you want to know more, you can go to the Microsoft Docs for more information, in the [Links](#links) section.

When referencing any of these Security Principals within the Domain, the SID will be used, and then 'looked up' through a Domain Controller, the Domain Controller will then return Security Principal related to the SID. Now, if the Domain Controller is not able to translate this SID into a Security Principal, I will just be displayed as the plain SID. Places where you 'often' see this, is when you go into the Security Pane of your objects in your Active Directory Users and Computers console, here it can be displayed either as just the SID, or as "Account Unknown(S-1-5-21-1234567890-123456789-1234567890-2535)".

### How do they become Unknown/Orphaned/Unresolvable?

Well, as some might have figured out, since the SID are used to reference a Security Principal, if someone were to delete the Security Principal, the SID becomes unresolvable, and hence also unknown and therefore an orphan. Active Directory do not remove all the references to an object, since it the lookup works the other way around, so it will look up the Security Principal when it encounter a reference (SID) to it, but Active Directory will not really keep track of where these references are, if you delete a user or security group, which has delegated permissions all over your Active Directory, you will leave a lot of orphans.

## Risk(s)

What are the risks associated with this? **Total Domain Domination!** is a possibility, since as an article from ADSecurity (link in [Links](#links) section) discribes it's possible to injecting a SIDHistory reference on an user, and thereby obtaining the permissions of the SID you inject. Normally you would use this feature to inject the SID of a Domain Admin, somthing like this are noticed pretty easy, but if you have a lot of unresolved SIDs in the AD, with different permissions, you could leverage these with anyone really noticing. You can even encounter unresolved SIDs in some Security Groups, so simply by using these, an attacker could get membership of privileged groups, without changing the groups, and thereby bypass the changing of a privileged group, and not triggering the auditing of changes in membership.

## Mitigation

### Documented processes on deleting objects in Active Directory

By simply remembering to remove groupmemberships of an Security Principal, you mitigate the former mentioned possibility of obtaining membership of an security group unnoticed, because by removing the memberships prior to deletion, the Security Principal is removed from the groups, not leaving an unresolved SID in the security groups, when it is deleted. So by having a documented process in creating and deleting users, and following it, you could actually mitigate a big part of the problem.

### Remember to remove the SIDHistory from your Security Principals

If you are a part of company that at some point in time have migrated either from one Active Directory to another, og if your company have merged with another company, and either migrated their Security Principals into your Active Directory, or the other way around, you should make sure that **ALL** the objects in your have had their SIDHistory cleared, since there no reason whatsoever, to not clearing the SIDHistory on the Security Principals in your Active Directory once a migration of Security Principals are complete.

### Audit your ACL's on a regular basis

Finally, and this is the one, that will make you cry, you should on a regular basis scan your Active Directory with a tool like AD ACL Scanner (link in the [Links](#links) section) and go through this list either manually or using a script. To identify all the ACL's of your Active Directory to both identify unresolved SIDs, but also to detect unwanted or unintented permission. When auditing the ACL's it's also worth looking for who Own's the different object, since the Owners have Delegate Permissions, and in migrated enviroments, the Owner is often a SID from the Active Directory, from where the object was migrated from.

## Toolbox

### Basic PowerShell commands

\>TO COME SHORTLY<

### Scripts / Modules

#### [ADObjectOwner](https://github.com/tomstryhn/ADObjectOwner)

A PowerShell module I have created, to help change the Owner of on or more Active Directory Objects. Keep in mind, that often are Permissions inherited, so the Owner of for instance an OU, are able to Delegate Permissions on that object, which could possible affect nested objects.

## Links

Be aware that due to the nature of Github, and the way links work, these links will not open in a new window, unless you press \<CTRL> or \<SHIFT> while clicking them.

#### <a href="https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/78eb9013-1c3a-4970-ad1f-2b1dad588a25">[MS-DTYP]: SID | Microsoft Docs</a>
Further reading on the Security Identifier (SID)
#### [ADSecurity](https://adsecurity.org/?p=1772)
Article on SID History injection
#### [AD ACL Scanner](https://github.com/canix1/ADACLScanner)
Great tool for auditing the ACL's in you Active Directory
