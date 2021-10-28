# Active Directory Unknown SIDs

Unknown SID, Orphaned SID or Unresolvable SID, all three terms cover the same issue, an issue that many AD Administrators, at some point have encountered and/or are battling with, not to mention the hassle to get them all cleared out. I will with along with this Guide supply some code and scripts that might help with this.

## What is an Unknown/Orphaned/Unresolvable SID?

To understand what it is, you need to have a little insigth, and first need to know what a SID is.

SID is an abbrevation for Security Identifier, below I display the SID for a ADUser, using PowerShell.

```PowerShell
PS C:\> (Get-ADUser Administrator).SID

BinaryLength AccountDomainSid                         Value                                       
------------ ----------------                         -----                                       
          28 S-1-5-21-1234567890-123456789-1234567890 S-1-5-21-1234567890-123456789-1234567890-500
```
> This is a sample SID for the domain administrator of our fictive AD. (Note: The builtin Administrator always ends with '-500')

A SID uniquely identifies security principals, primarily *Computers*, *Groups* and *Users*, there are a few other objecttypes in AD that actually have objectsid's but for now we will focus on these primary three object types, since they are also the primary reason for our troubles. As seen on the PowerShell example above, the SID consists of two primary parts, the AccountDomainSid, which identifies the domain to which the object belongs, and the last part, which you could call the serial number for the security principal, and which is unique in the domain. There are a few more technical details in regard to the SID, but if you want to know more, you can go to the Microsoft Docs for more information.

When referencing any of these Security Principals within the Domain, the SID will presented, and then 'looked up' by the through the Domain Controller, who will present the Identity related to the SID. Now, if the Domain Controller is not able to translate this SID into a Security Principal or Identity, I will just be displayed as the plain SID. Places where you normally see this is when you go into the Security Pane of your objects in your Active Directory Users and Computers console, here all it can be displayed either as just the SID, or as Unknown.
