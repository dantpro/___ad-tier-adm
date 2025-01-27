# Index
- [Index](#index)
- [Introduction](#introduction)
    - [Steps](#steps)
    - [Links](#links)
    - [Requirements](#requirements)
- [Create GPO and link to OU](#create-gpo-and-link-to-ou)
    - [Create GPO](#create-gpo)
    - [Create settings for the GPO](#create-settings-for-the-gpo)
    - [Link OU](#link-ou)
    - [Conclusion step 1](#conclusion-step-1)
- [Check for new hosts](#check-for-new-hosts)
    - [Powershell script](#powershell-script)
- [Setup Scheduled Task](#setup-scheduled-task)
    - [Create Service Account](#create-service-account)
    - [Create new scheduled task](#create-new-scheduled-task)
- [Conclusion](#conclusion)
# Introduction
**Managing local admins in Active Directory can be a challenging task. Especially when you want to give single users local admin permissions on a single asset. Typically this would require adding the user account to the local administrator group, but this is difficult to audit and typically the permissions are not removed when no longer needed.**  
   
To manage the local admin permissions from Active Directory you can work with group policies, but with this approach you cant assign a single user to a single asset. Every asset linked to the GPO will have the same configuration. 

In this guide I explain how to create a GPO assigning dynamic groups to the local administrator group on an asset. This does require a powershell script and a scheduled task to be created on a domain controller.  

There are multiple benefits to this approach:
- Dynamic Local admin permissions.
- Settings are reverted every time the GPO is applied, manually added members are automatically removed.
- Easy audit on specific assets from Active Directory

This guide aims to use Powershell to complete the task, this 

### Steps

- Create GPO and link to OU
- Setup scheduled task and powershell script

### Links
[Powershell Policy Editor](https://pspeditor.azurewebsites.net/)  
[Group Creation Script](https://github.blabla)

### Requirements

- AD domain
- OU containing computer objects
- Account with Domain Administrator privileges

In this guide I use 

# Create GPO and link to OU
We need to create a Group Policy object and link it to the OU containing the assets we want to control, in this guide we use the Tier 1 Service OU.
> OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local

### Create GPO

First we create a new GPO containing the settings to add members to the local admin group. In this example we create a GPO for all tier 1 servers. To achieve this perform the following steps:

- Create new GPO by running the following powershell command
```
PS C:\Users\Administrator> New-GPO -Name 'C_ADM_Local Admin tier 1' -Comment 'Tier 1 Local administrators'


DisplayName      : C_ADM_Local Admin tier 1
DomainName       : test.local
Owner            : TEST\Domain Admins
Id               : eb5b12dd-0213-4c12-8e44-b2f281a2ffbc
GpoStatus        : AllSettingsEnabled
Description      : Tier 1 Local administrators
CreationTime     : 11/22/2024 3:09:59 PM
ModificationTime : 11/22/2024 3:09:59 PM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 0, SysVol Version: 0
WmiFilter        :
```
> This creates the GPO and makes it available in the Group Policy Objects folder, but does not yet links it to any OU.

### Create settings for the GPO
Now we need to populate the GPO with settings. Follow the following steps:
- In Group Policy management console open the folder *Group Policy Objects* and right click the GPO we just created.
- Click edit settings and a new window opens with the settings of the GPO.
- Go to *computer configuration/Preferences/Control Panel Settings/Local Users and Groups* right click and choose new -> Local Group.
- Set *Action* to **Update**, *Group name* to **Administrators (built-in)** and check the check box for *Delete all member users*, but **NOT** for *Delete all member groups* or all default groups will also be deleted.
- At the *Members* section click on **Add...** and type **TEST\%COMPUTERNAME%_LocalAdm** with action *Add to this group* and click OK. Do not use the **...** button, this will allow you to search for a group, but obviously a group with this name does not exist.
- Under the tab *Common* you can enter a description if you want, but no other settings have to be changed.
- Click ok to save all the settings and you should see a new group called *Administrators (built-in) with a yellow triangle.

The GPO is finished and can be linked to the OU now.

### Link OU

To link the OU, get the *distinguishedName* attribute value if the OU.
- Open **User and Computers** management console.
- Make sure advanced features are enabled in the View menu.
- Go the the OU where you want to link the GPO and right click on it and choose properties.
- Go to the attributes tab and find the field for *distinguishedName* and copy the value.

In this guide we use the Tier 1 OU within the Service OU. The *distinguishedName* would be **OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local**. Now open a Powershell console with administrator privileges and run the following command:

```
PS C:\Users\Administrator> New-GPLink -Guid eb5b12dd-0213-4c12-8e44-b2f281a2ffbc -Target 'OU=Tier 1,OU=Service,OU=Corp,DC=test,DC=local'


GpoId       : eb5b12dd-0213-4c12-8e44-b2f281a2ffbc
DisplayName : C_ADM_Local Admin tier 1
Enabled     : True
Enforced    : False
Target      : OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local
Order       : 1
```

As you can see we used the ID value of when we created the GPO to identify which GPO we want to link and we use the *distinguishedName* value as target. 

### Conclusion step 1
This concludes the first step *Create GPO and link to OU*. We can continue to the next step, but first I want to make some remarks.

Because we are using the %COMPUTERNAME% variable as group name when the GPO is processed on a client it will look for a group containing the hostname of the client. If this group does not exist it will have a resolution error, this is a non blocking error and all other GPO processing will continue as normal. Because the group does not exist by default we have to regularly check if there are new hosts added to the specified OU. That will be the topic of the next step.

# Check for new hosts
To check for new hosts we will use a Powershell script and configure an scheduled task for this script. This task will run every 1 hour, but obviously this can be configured as you wish.

First thing to do is to decide where you want to create the groups, this script does not set permissions on the group and therefore the group will inherit permissions from the delegation model. 

> For this guide we use the following location for the LocalAdm groups: **OU=LocalAdm_Tier1,OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local**

### Powershell script
The script will do two tasks:
- Check for each host if a LocalAdm group exists.
- Create a new LocalAdm group if a host does not have a group.

> You could also choose to hardcode the *SearchBase* and the *GroupPath*, but by using parameters the script can be reused for different locations allowing to process multiple locations separately with the same script. 

Execute the script with the following command to test it before setting up the scheduled task.
```
PS C:\Temp> .\Create-LADMGroups.ps1 -SearchBase 'OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local' -GroupPath 'OU=LocalAdm_Tier1,OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local'


DistinguishedName : CN=ComputerObject1,OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local
DNSHostName       : 
Enabled           : True
Name              : ComputerObject1
ObjectClass       : computer
ObjectGUID        : 09b18963-6323-466c-98a6-fd6d827cf0a7
SamAccountName    : COMPUTEROBJECT1$
SID               : S-1-5-21-783641718-2592122621-1651618145-1267
UserPrincipalName : 

DistinguishedName : CN=ComputerObject2,OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local
DNSHostName       : 
Enabled           : True
Name              : ComputerObject2
ObjectClass       : computer
ObjectGUID        : 69115ac7-08f8-4d41-8465-d26e147fbb25
SamAccountName    : COMPUTEROBJECT2$
SID               : S-1-5-21-783641718-2592122621-1651618145-1268
UserPrincipalName : 
```

> You should run the script on a test OU first with a limited number of computer objects. When confident the script is working well you can move to production OU's.

After the script was manually executed verify the existence of the groups. If the groups are created we can move to the next step.

# Setup Scheduled Task
In this step we will setup the scheduled task with the help of Powershell. We need to create an new service account to perform this action. 

### Create Service Account
In this step we assume the domain is configured with a tiering model and consists of 3 tiers. The service account should have enough permissions on the OU where the LocalAdm groups are located. It should be able to create groups in the OU. If you have been following this guide: [Active Directory Tiering Model deployment with powershell](https://freelance.fhs7.nl/blog/active-directory-tiering-model-deployment-with-powershell) you can create a tier 1 service account and make it member of the **GRP_Tier 1 ADM Access Control_MANAGE** access group. This allows the service account to be able to create new groups in the Access Control container in the tier 1 OU. 

### Create new scheduled task
In this step we assume you stored the script in C:\Scripts. Open powershell prompt and type the following commands:

```
PS C:\> $action = New-ScheduledTaskAction -Execute "Powershell" -Argument -NoProfile -ExecutionPolicy Bypass -file 'C:\Scripts\Create-LADMGroups.ps1'
PS C:\> $trigger1 = New-ScheduledTaskTrigger -Daily -At 01:00
PS C:\> $trigger2 = New-ScheduledTaskTrigger -Once -At 01:00 `
            -RepetitionInterval (New-TimeSpan -Minutes 15) `
            -RepetitionDuration (New-TimeSpan -Hours 23 -Minutes 55)
PS C:\> $principal = "Test\SA-LADMGroup"
PS C:\> $settings = New-ScheduledTaskSettingsSet
PS C:\> $task = New-ScheduledTask -Action $action -Principal $principal -Trigger $trigger -Settings $settings
PS C:\> Register-ScheduledTask T1 -InputObject $task
```

# Conclusion



