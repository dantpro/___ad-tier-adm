# Index
- [Index](#index)
- [Introduction](#introduction)
    - [Tiering Model Refresh](#tiering-model-refresh)
    - [Tier 0 Domain](#tier-0-domain)
    - [Tier 1 Services](#tier-1-services)
    - [Tier 2 End Users](#tier-2-end-users)
    - [The three Commandments](#the-three-commandments)
- [Implementing tier 0 protection](#implementing-tier-0-protection)
    - [Authentication Policies](#authentication-policies)
    - [Deep Dive in Authentication Policies](#deep-dive-in-authentication-policies)
    - [Implementing Tier 0 Authentication policy](#implementing-tier-0-authentication-policy)
    - [Prerequisites Kerberos Authentication Policies](#prerequisites-kerberos-authentication-policies)
    - [Authentication Policy Settings](#authentication-policy-settings)
- [Tier 0 Admin Logon Flow PAWs are a Must](#tier-0-admin-logon-flow-paws-are-a-must)
    - [PAWs](#paws)
    - [The cloud](#the-cloud)

# Introduction
The AD Administrative Tier Model prevents escalation of privilege by restricting what Administrators can control and where they can log on. In the context of protecting Tier 0, the latter ensures that Tier 0 credentials cannot be exposed to a system belonging to another Tier (Tier 1 or Tier 2).

### Tiering Model Refresh
The tiering model is the best practice Microsoft promotes since the *red forest* approach was no longer considered safe due to vulnerabilities discoverd in the trust relationships between domains. The tiering model typically consists of 3 layers and provides isolation for each layer.

> More layers can be added if desired, for example to create an separate application / data layer for company sensitive data.

### Tier 0 Domain
The first tier is your domain tier and also your most sensitive tier where most effort is put in to protect it. This is where your domain controllers reside and your certificate infrastructure (PKI).

### Tier 1 Services
In the second tier all your services will reside. Think of web servers, file servers or other services. These are the services used by your end users.

### Tier 2 End Users
The last tier is for the end users and their devices. This tier is the most exposed to the outside world and consequently the most dangerous tier as well. It means we need to be careful what we access with these devices and prevent using these devices to manage our sensitive assets within our network.

### The three Commandments
There are 3 rules, or call them commandments which are crucial for a tiering model to function. The three commandments are:
- **Credentials from a higher-privileged tier** (e.g. Tier 0 Admin or Service account) **must not be exposed to lower-tier systems** (e.g. Tier 1 or Tier 2 systems).
- **Lower-tier credentials can use services provided by higher-tiers, but not the other way around**. E.g. Tier 1 and even Tier 2 system still must be able to apply Group Policies.
- **Any system or user account that can manage a higher tier is also a member of that tier, whether originally intended or not.**  

When you follow these 3 rules you can reduce your attack surface a lot. The picture below explains how these 3 rules work.  
<sub>*Microsoft Tiering Model*</sub>  
![Tiering Model](https://techcommunity.microsoft.com/t5/s/gxcuf89792/images/bS00MDUyODUxLTU1MDI2Mmk4RTYwNjEwMzI1QUM3RTJB?image-dimensions=450x450&revision=21)  
<sub>*Src: Microsoft Community Hub*</sub> 

# Implementing tier 0 protection
In most guides for deploying an AD tiering model you will read about complicated GPO setups that prevent higher tiered administrative accounts to login on lower tiered assets. This approach comes with a couple of downsides.
- Group policies can be bypassed by administrators
- Only domain joined assets are affected  

### Authentication Policies
To counter this there is a solution called *auhentication policies*, authentication Policies provide a way to contain high-privilege credentials to systems that are only pertinent to selected users, computers, or services. With these capabilities, you can limit Tier 0 account usage to Tier 0 hosts. That’s exactly what we need to achieve to protect Tier 0 identities from credential theft-based attacks.  

> With Kerberos Authentication Policies you can define a claim which defines where the user is allowed to request a Kerberos Granting Ticket from.

### Deep Dive in Authentication Policies
Authentication policies work on *Kerberos* level, allowing to set restriction on authentication protocol level. More specifically, the Kerberos extension called FAST (Flexible Authentication Secure Tunneling) is used by Authentication policies. 

**FAST (Flexible Authentication Secure Tunneling)**  
FAST provides a protected channel between the Kerberos client and the KDC for the whole pre-authentication conversation by encrypting the pre-authentication messages with a so-called armor key and by ensuring the integrity of the messages. By default Kerberos Armoring is disabled and must be enabled using group policies and once enabled it provides the following functionality:  
- Protection against offline dictionary attacks. Kerberos armoring protects the user’s pre-authentication data (which is vulnerable to offline dictionary attacks when it is generated from a password).
- Authenticated Kerberos errors. Kerberos armoring protects user Kerberos authentications from KDC Kerberos error spoofing, which can downgrade to NTLM or weaker cryptography.
- Disables any authentication protocol except Kerberos for the configured user.
- Compounded authentication in Dynamic Access Control (DAC). This allows authorization based on the combination of both user claims and device claims.  

The last bullet point provides the basis for the feature we plan to use for protecting Tier 0: Authentication Policies.  

By using authentication policies to restrict user logon from specific hosts, we require the Domain Controller (Key Distribution Center (KDC)) to validate the hosts identity. When used with Kerberos Armoring, the KDC is provided with the TGT of the host from which the user is authenticating. That’s what we call an armored TGT, the content of which is used to complete an access check to determine if the host is allowed.  
<sub>*Kerberos Authentication Flow*</sub>  
![Kerberos Authentication](https://techcommunity.microsoft.com/t5/s/gxcuf89792/images/bS00MDUyODUxLTU1MDI3N2kyODU1MzE0OTgzMEVGRTE4?revision=21)  
<sub>*Src: Microsoft Community Hub*</sub>  
Kerberos armoring logon flow (high level)
- The computer gets a TGT during the computer authentication to the domain.
- The user logs on to the computer.
  - An unarmored AS-REQ for a TGT is sent to the KDC.
  - The KDC queries for the user account in Active Directory and determines if it is configured with an Authentication Policy that restricts initial authentication that requires armored requests.
  - The KDC fails the request and asks for Pre-Authentication.
  - Windows detects that the domain supports Kerberos armoring and sends an armored AS-REQ to retry the sign-in request.
  - The KDC performs an access check by using the configured access control conditions and the client operating system’s identity information in the TGT that was used to armor the request. If the access check fails, the domain controller rejects the request.
- If the access check succeeds, the KDC replies with an armored reply (AS-REP) and the authentication process continues. The user now has an armored TGT.  

The difference with a normal kerberos authentication is that a users TGT also includes the computers identity. After this the rest of the process looks similar when requesting access to resources, except that the users armored TGT is used for protection and restriction.

### Implementing Tier 0 Authentication policy

To contain Tier 0 accounts (Admins and Service accounts) the following steps are required:
- Enable Kerberos Armoring (aka FAST) for DC's and tier 0 computer objects.
- You need to have an OU structure in place that supports tiering of Active Directory.
- Groups for Tier 0 accounts and Tier 0 computers should be in place.
- Frequently update Authentication Policies to ensure that any new T0 Admin, T0 Service account or T0 computer is covered by the Authentication policy.
- Configure an Authentication Policy with the following parameters and enforce the Kerberos Authentication policy:  

| (User) Accounts | Conditions (Computer accounts/groups) | User Sign On |
|:----------------|:--------------------------------------|:-------------|
| T0 Admin accounts | (Member of each({ENTERPRISE DOMAIN CONTROLLERS}) Or Member of any({Tier 0 computers (TEST\Tier 0 computers)})) | Kerberos only |

<sub>*Screenshot Authentication Policy*</sub>  
![Authentication Policy](https://techcommunity.microsoft.com/t5/s/gxcuf89792/images/bS00MDUyODUxLTU1MDI4OWk2MDZBQ0ZDMEI0Mzg2OEVB?image-dimensions=750x750&revision=21)  

Find more details about how to create Authentication Policies at [Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/how-to-configure-protected-accounts#create-a-user-account-audit-for-authentication-policy-with-adac).  

### Prerequisites Kerberos Authentication Policies
Kerberos Authentication Policies were introduced in Windows Server 2012 R2, hence a Domain functional level of Windows Server 2012 R2 or higher is required for implementation.

### Authentication Policy Settings
**Require rolling NTLM secret for NTLM authentication**  
Configuration of this feature was moved to the properties of the domain in Active Directory Administrative Center. When enabled, for users with the “Smart card is required for interactive logon” checkbox set, a new random password will be generated according to the password policy. See https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/whats-new-in-credential-protection#rolling-public-key-only-users-ntlm-secrets for more details.

**Allow NTLM network authentication when user is restricted to selected devices**   
This setting is not recommended, allowing NTLM authentication reduces the capabilities of restricting access through Authentication Policies. Next to this, its recommended to place privileged user accounts in the **Protected Users** security group which is designed to harden privileged accounts and introduces a set of protection mechanisms, one of which is making NTLM Authentication impossible for the members. See https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/authentication-policies-and-authentication-policy-silos#about-authentication-policies for more details.  

**Have Breakglass Accounts in place**  
Break Glass accounts are emergency access accounts used to access critical systems or resources when other authentication mechanisms fail or are unavailable. In Active Directory, Break Glass accounts are used to provide emergency access to Active Directory in case normal T0 Admin accounts do not work anymore, e.g. because of a misconfigured Authentication Policy.

# Tier 0 Admin Logon Flow PAWs are a Must  
Attackers can make their way into environments even with MFA measures in place, through for example open RDP connections when an admins computer is compromised. To protect your self from these kind of attacks you can make use of Privileged Access Workstations (PAWs). This has been a recommendation of Microsoft for years. 

### PAWs
PAWs are workstations that a specifically used for Tier 0 administration and nothing else. These workstations only accept Tier 0 accounts to login.  

> A lot of organizations choose to setup a protects RDS farm to replace PAWs, modern PAM solutions can provide session redirection where a Tier 0 admin is automatically connected to a Tier 0 RDS Session Host. You need to take extra effort in protecting your domain when not making use of PAWs.

**PAWs vs Normal workstations**
The advantage of using PAWs compared to normal workstations is that normal workstations have a bigger risk of being compromised. For example a laptop used for Email, Meetings and other office tasks is constantly exposed to potential threats. A PAW is only used for Domain administrative tasks and therefore the risk is much lower for it to be compromised.

The classic logon flow is with a Domain Joined PAW, see picture below. 

<sub>*Tier 0 Logon flow*</sub>  
![Tier 0 Logon flow](https://techcommunity.microsoft.com/t5/s/gxcuf89792/images/bS00MDUyODUxLTU1MDI5NmlERkMzNzM2RUVDNEVEQkJG?revision=21)   

### The cloud
The solution above is straightforward but does not provide any modern cloud-based security features such as:
- Multi factor Authentication
- Conditional Access
- Identity Protection

There are more benefits to using Azure and Intune, but these 3 points are relevant to the topic.

**Protecting Tier 0 the Cloud way**  
The cloud way of Protecting Tier 0 is by using an *Intune-managed PAW* and *Azure Virtual Desktop*, this approach is easy to implement and perfectly teams modern protection mechanisms with on-prem Active Directory. 

![Tier 0 Logon Flow Cloud way](https://techcommunity.microsoft.com/t5/s/gxcuf89792/images/bS00MDUyODUxLTU1MDI5N2k4OTUyQUExRDA4NzE4OEFC?revision=21)  

Logon to the AVD is restricted to come from a compliant PAW device only, Authentication Policies do the rest.