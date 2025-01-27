# Index

- [Index](#index)
- [Introduction](#introduction)
    - [Links](#links)
    - [Requirements](#requirements)
    - [Steps](#steps)
- [Deploy OU / Container Structure](#deploy-ou--container-structure)
    - [Prerequisites](#prerequisites)
    - [Run script](#run-script)
- [Creating groups](#creating-groups)
    - [Prerequisites](#prerequisites-1)
    - [Run Script](#run-script-1)
- [Set Group Memberships](#set-group-memberships)
    - [Requirements](#requirements-1)
    - [Run Script](#run-script-2)
- [Creating Password Settings Objects](#creating-password-settings-objects)
    - [Prerequisites](#prerequisites-2)
    - [Run Script](#run-script-3)
- [Creating Group Policy Objects](#creating-group-policy-objects)
    - [Naming Convention](#naming-convention)
    - [Admin Levels](#admin-levels)
    - [Run Script](#run-script-4)
- [Creating AD roles and permissions](#creating-ad-roles-and-permissions)
    - [Requirements](#requirements-2)
    - [Run Script](#run-script-5)
- [Deploy Users](#deploy-users)
    - [Requirements](#requirements-3)
    - [Run Script](#run-script-6)
- [Conclusion](#conclusion)

# Introduction

**This set of PowerShell scripts provides a streamlined way to deploy a tiered model in Active Directory. It’s designed for both existing AD environments where it creates a new top-level OU and implements the tiering model and brand-new AD setups, allowing you to establish a tiered structure right from the start.**
<br></br>
It’s recommended to first test the deployment in a lab environment. This will help you become familiar with the scripts and associated files, ensuring a smooth and confident rollout in production.

### Links

[Download scripts from GitHub](https://github.com/Oyon84/AD-Tier-Administration/archive/refs/heads/master.zip)   
[GitHub Repo](https://github.com/Oyon84/AD-Tier-Administration)

### Requirements
- AD domain
- Domain admin level permissions
- Text editor to edit the input files (csv / json / log)
- Powershell client, can be visual studio code or Powershell ISE or other preference

### Steps
1. Deploy OU / Container Structure (Create-Structure.ps1)
2. Creating groups (Create-Groups.ps1)
3. Deploy group memberships for roles (Add-MembersToGroups.ps1)
4. Creating Password Settings Objects (Create-PSOs.ps1)
5. Creating Group Policy Objects (Create-GPOs.ps1)
6. Creating AD roles and permissions (Create-ACEs.ps1)
7. Creating User for each tier (Create-Users.ps1)

# Deploy OU / Container Structure  
**Script:** (Create-Structure.ps1)  
First step is to create a new OU structure. This script will deploy a new Top Level OU called **corp** and deploys a new OU structure in the corp OU. This will contain OU's for all tiers and assets and this forms the basis of the tiering model. This script also deploys a set of containers for the following assets:
- Roles Tier 0, 1, 2
- Access Control Tier 0, 1, 2
- Security Groups

The script will perform a check the object after its created, if something failed it will be shown on the console with Red text displaying the error.

### Prerequisites 
- Rename user locations in **Create-Structure.ps1** to something sensible for your organization. Currently marked as Location 1, 2, 3, 4 and 5.
- Rename path to your domain. Current value is "DC=test,DC=local" standing for test.local as domain name. Use your text editor to find DC=test,DC=local and replace with DC=yourdomain,DC=local
- Create any extra OU's 

### Run script
Next step is to run the script. Output should look similar to the following: 

```
PS C:\Users\Administrator> C:\Temp\Create-Structure.ps1
Creating OU Corp in DC=test,DC=local
Creating OU Users in OU=Corp,DC=test,DC=local
Creating OU Service in OU=Corp,DC=test,DC=local
Creating OU Administration in OU=Corp,DC=test,DC=local
Creating OU Computers in OU=Corp,DC=test,DC=local
Creating OU Tier 2 in OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Tier 1 in OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Tier 0 in OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Terminal servers in OU=Computers,OU=Corp,DC=test,DC=local
Creating OU PKI in OU=Tier 0,OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Shared in OU=Tier 2,OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Desktops in OU=Tier 2,OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Laptops in OU=Tier 2,OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Network in OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Application in OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local
Creating OU File in OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Update in OU=Tier 1,OU=Computers,OU=Corp,DC=test,DC=local
Creating OU Delft in OU=Users,OU=Corp,DC=test,DC=local
Creating OU Location2 in OU=Users,OU=Corp,DC=test,DC=local
Creating OU Location3 in OU=Users,OU=Corp,DC=test,DC=local
Creating OU Location4 in OU=Users,OU=Corp,DC=test,DC=local
Creating OU Location5 in OU=Users,OU=Corp,DC=test,DC=local
Creating OU Offsite in OU=Users,OU=Corp,DC=test,DC=local
Creating OU Contractors in OU=Users,OU=Corp,DC=test,DC=local
Creating OU Tier 0 in OU=Service,OU=Corp,DC=test,DC=local
Creating OU Tier 1 in OU=Service,OU=Corp,DC=test,DC=local
Creating OU Users in OU=Tier 0,OU=Service,OU=Corp,DC=test,DC=local
Creating OU Users in OU=Tier 1,OU=Service,OU=Corp,DC=test,DC=local
Creating OU Tier 0 in OU=Administration,OU=Corp,DC=test,DC=local
Creating OU Tier 1 in OU=Administration,OU=Corp,DC=test,DC=local
Creating OU Tier 2 in OU=Administration,OU=Corp,DC=test,DC=local
Creating OU Users in OU=Tier 0,OU=Administration,OU=Corp,DC=test,DC=local
Creating OU Users in OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local
Creating OU Users in OU=Tier 2,OU=Administration,OU=Corp,DC=test,DC=local
Creating Container Security groups in OU=Corp,DC=test,DC=local
Creating Container Access Control in CN=Security groups,OU=Corp,DC=test,DC=local
Creating Container Roles in CN=Security groups,OU=Corp,DC=test,DC=local
Creating Container Access Control in OU=Tier 0,OU=Administration,OU=Corp,DC=test,DC=local
Creating Container Roles in OU=Tier 0,OU=Administration,OU=Corp,DC=test,DC=local
Creating Container Access Control in OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local
Creating Container Roles in OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local
Creating Container Access Control in OU=Tier 2,OU=Administration,OU=Corp,DC=test,DC=local
Creating Container Roles in OU=Tier 2,OU=Administration,OU=Corp,DC=test,DC=local
Creating Container Access Control in OU=Tier 1,OU=Service,OU=Corp,DC=test,DC=local
Creating Container Access Control in OU=Tier 0,OU=Service,OU=Corp,DC=test,DC=local

PS C:\Users\Administrator> 
```

Fix any error you come across. And the Structure deployment is done.

# Creating groups  
**Script:** (Create-Groups.ps1)  
Next step is to create groups that will be used for different purposes. The "**group-creation.csv**" file contains all default groups needed for a basic tiering model. You can add extra groups as you desire in the same format.

### Prerequisites 
- Again you will have to correct the domain name, but this time in the CSV file. Open the CSV file in a text editor and replace the same values as in step 1 to match you domain.
- Store the script file (Create-Groups.ps1) and the CSV file (group-creation.csv) in the same folder. The script looks for the CSV file in the root where the script is executed. If you use ISE, go to the location of the script, for example: ```cd c:\temp```
- Add any custom groups you want to deploy as well to the CSV file, see example:
```
Name,Path,GroupScope,GroupCategory
COMPTR_Default Container_CREATE,"CN=Access Control,OU=Tier 2,OU=Administration,OU=Corp,DC=test,DC=local",DomainLocal,Security
```

### Run Script
Now you can run the script, the output should give you information about the groups successfully created and als the ones that failed to be created. Check for errors in the output, correct them as needed and run the script again. Groups that already exist will be skipped, the ones that failed will be created if the issue is resolved.

```
If the script gives an error that it can't find the CSV file check if the script and csv are in the same folder
and your console location is that folder.
```
Once all groups are created and the script only give "skipping" messages you can continue with the next step.

# Set Group Memberships  
**Script:** (Add-MembersToGroups.ps1)  
Here we add the role groups to the correct permission groups. This is done with the input from a CSV file.

### Requirements
A input CSV file is needed called group-membership. This file contains two collumns:

| Group              | addto           |
|:-------------------|:----------------|
| "ROLE_T0_AD Admin" | "Domain admins" |
| "ROLE_T0_Global GPO Admin" | "GPO_Tier 0 Global GPO_MOD" |

Modify the file to your needs.

### Run Script
When the CSV file is ready the script can be run. It will first check if the role is not yet a member of the group in the "addto" column. If so it will skip and go to the next.

```
PS C:\Temp> C:\Temp\Add-MembersToGroups.ps1
Adding ROLE_T0_SCHM Admin to Schema admins
Adding ROLE_T0_ENTRP Admin to Enterprise admins
Adding ROLE_T0_AD Admin to Domain admins
Adding ROLE_T0_Global GPO Admin to GPO_Tier 0 Global GPO_MOD
Adding ROLE_T0_PKI Server Admin to SRV_Tier 0 PKI SRV_MOD
Adding ROLE_T0_PKI Server Admin to SRV_Tier 0 PKI SRV_RENAME
...
```
*Warning if membership already exists*
```
PS C:\Temp> C:\Temp\Add-MembersToGroups.ps1
WARNING : Group  ROLE_T0_Global GPO Admin  is already a member of GPO_Tier 0 Global GPO_MOD
```

Look for any errors and correct them where needed.

# Creating Password Settings Objects  
**Script:** (Create-PSOs.ps1)  
PSO's are objects that enforce password requirements. Using PSO's gives the ability to set different scopes of password requirements per tier. This script does not require an input file.

### Prerequisites
Check the script file to validate the password settings are according to your organizations need. There are 5 objects defined in the script.   

  | Scope                  | Object                | Applies to                     | Max Age | Min Length | History Count | Precedence |
  |------------------------|-----------------------|--------------------------------|--------:|-----------:|--------------:|-----------:|
  | Tier 0 Admin           | PSO_AT0_Administrator | PSO_Tier 0 ADM Users_APPLY     | 120     | 20         | 20            | 10         |
  | Tier 1 Admin           | PSO_AT1_Administrator | PSO_Tier 1 ADM Users_APPLY     | 120     | 20         | 20            | 20         |
  | Tier 2 Admin           | PSO_AT2_Administrator | PSO_Tier 2 ADM Users_APPLY     | 120     | 16         | 20            | 30         |
  | Tier 0 Service Account | PSO_ST0_Service User  | PSO_Tier 0 Service Users_APPLY | 10000   | 100        | 20            | 10         |
  | Tier 1 Service Account | PSO_ST1_Service User  | PSO_Tier 1 Service Users_APPLY | 10000   | 100        | 20            | 20         |
  

> Note: For all PSO's complexity is turned on requiring complex passwords

Validate the settings with your organizations security policies, if you don't have any the default settings are a good starting point.

### Run Script
When all settings are correct you can run the script. This will create the Password Settings Objects and link them to the correct groups. This will enforce the password policies to the right accounts when all scripts are deployed.


> Note: This script has no output, so if the script finished without any output it has run successful.
To verify if objects are created you can run the script again, and it should tell you the objects already exist.

# Creating Group Policy Objects  
**Script:** (Create-GPOs.ps1)  
The following script does 3 things:
- Create GPO according input given
- Set permission based on admin level
- Link GPO to correct OU
What this script does not do is setting the settings for these GPO.

### Naming Convention
This script checks for naming compliance and uses the following rules
- Should start with C | U | UC | CU
- Policy type in capitals, for example: SEC, ADM, PRINTER, Max 15 chars
- Policy name

During execution of the script with both methods the naming compliance is checked and if fails the scripts breaks.

### Admin Levels
The script defines the following admin levels. These translate to groups created prior and grant permissions on GPO's. Below an overview of the default admin levels and linked groups.

| Admin Level | Group |
|:------------|:------|
| Tier0Global                       | "GPO_Tier 0 Global GPO_MOD" |
| Tier0PKIServers                   | "GPO_Tier 0 PKI SRV_MOD" |
| Tier0PrivilegedAccessWorkstations | "GPO_Tier 0 Privileged Access Workstations_MOD" |
| Tier1AllServers                   | "GPO_Tier 1 All SRV_MOD" |
| Tier1ADMUsers                     | "GPO_Tier 1 ADM Users_MOD" |
| Tier1ServiceUsers                 | "GPO_Tier 1 Service_MOD" |
| Tier1NetServers                   | "GPO_Tier 1 Network SRV_MOD" |
| Tier1AppServers                   | "GPO_Tier 1 Application SRV_MOD" |
| Tier1FileServers                  | "GPO_Tier 1 File SRV_MOD" |
| Tier1UpdateServers                | "GPO_Tier 1 Update SRV_MOD" |
| Tier1TermServers                  | "GPO_Tier 1 Terminal SRV_MOD" |
| Tier1PrivilegedAccessWorkstations | "GPO_Tier 1 Privileged Access Workstations_MOD" |
| Tier2Workstations                 | "GPO_Tier 2 Workstations_MOD" |
| CorporateUsers                    | "GPO_Corporate Users_MOD" |

If you have changed the group names in step 2, that should also be changed in this step. 

### Run Script
The script has two excution methods.
- File (Use CSV input file)
- Interactive (Use console to enter GPO name and admin level)

For initial deployment its recommended to use the File input method, check the CSV file and adjust to requirements for the organization.
```
PS C:\Temp> .\Create-GPOs.ps1 C:\Temp\gpo-list.csv
```
If the script has run successful it should show the following output for each GPO in the CSV file.
```
Creating GPO C_PWR_Desktop power management
DisplayName      : C_PWR_Desktop power management
DomainName       : test.local
Owner            : TEST\Domain Admins
Id               : cdb51cfc-7158-4829-8a59-03b10b7e5320
GpoStatus        : AllSettingsEnabled
Description      : 
CreationTime     : 11/14/2024 12:17:04 PM
ModificationTime : 11/14/2024 12:17:04 PM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 0, SysVol Version: 0
WmiFilter        : 


Setting permission to GPO C_PWR_Desktop power management
DisplayName      : C_PWR_Desktop power management
DomainName       : test.local
Owner            : TEST\Domain Admins
Id               : cdb51cfc-7158-4829-8a59-03b10b7e5320
GpoStatus        : AllSettingsEnabled
Description      : 
CreationTime     : 11/14/2024 12:17:04 PM
ModificationTime : 11/14/2024 12:17:04 PM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 0, SysVol Version: 0
WmiFilter        : 


Linking GPO C_PWR_Desktop power management to OU OU=Desktops,OU=Tier 2,OU=Computers,OU=Corp,DC=test,DC=local

DisplayName   : C_PWR_Desktop power management
GpoId         : cdb51cfc-7158-4829-8a59-03b10b7e5320
Enabled       : True
Enforced      : False
Order         : 1
Target        : OU=Desktops,OU=Tier 2,OU=Computers,OU=Corp,DC=test,DC=local
GpoDomainName : test.local
```
All GPO objects are deployed and linked to the correct OU. Next step is to configure settings for the GPO's, but that part is not covered in this guid.

# Creating AD roles and permissions  
**Script:** (Create-ACEs.ps1)  
This script deploys permissions for different parts of active directory and assigns them to the roles deployed in earlier steps. These permissions are in the form of ACE's (access control entry) and can allow or deny access to objects. Dry-runs are supported to check for errors before applying the ACE's for real.

### Requirements
This script requires two input files in JSON format.
- ACESkel.json | ACE skeleton file defining ACE type, e.g. UnlockUser
- Permissions.json | Defining access groups and ACE per OU or CN

*ACESkel.json*
```
{
    "UnlockUser": [
        {
            "Type": "SpecificPermission",
            "ACE": {
                "ActiveDirectoryRights": "ReadProperty,WriteProperty",
                "AccessControlType": "Allow",
                "InheritedObjectTypes": [
                    "lockoutTime"
                ],
                "InheritanceType": "Descendents",
                "ObjectName": "user"
            }
        }
    ]
}
```
*Permissions.json*
```
[
    {
        "DistinguishedName": "OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local",
        "ACEs": [
                    {
                        "Group": "USR_Tier 1 ADM Users PW_RST",
                        "Permissions": [
                        "ResetPassword"
                        ]
                    }
                ]
    }
]
```
Change permission.json to your requirements, you can find example files in the GitHub repo.

### Run Script
Run the script without any parameters to do a dry-run. This will go through the whole script and checks the two input files, but will not update any ACL (access control list). 
```
PS C:\Temp> .\Create-ACEs.ps1
Set-ACL -Path AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local -AclObject System.DirectoryServices.ActiveDirectorySecurity -Verbose
Set-ACL -Path AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local -AclObject System.DirectoryServices.ActiveDirectorySecurity -Verbose
Set-ACL -Path AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local -AclObject System.DirectoryServices.ActiveDirectorySecurity -Verbose
Set-ACL -Path AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local -AclObject System.DirectoryServices.ActiveDirectorySecurity -Verbose
Set-ACL -Path AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local -AclObject System.DirectoryServices.ActiveDirectorySecurity -Verbose
Set-ACL -Path AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local -AclObject System.DirectoryServices.ActiveDirectorySecurity -Verbose
...
```
When the script has finished and errors are found a log file will be created *ACEDelegation.log* in the location you run the script from. Verify the errors and correct them before you run the script with the LIVE parameter.
```
PS C:\Temp> .\Create-ACEs.ps1 LIVE
VERBOSE: Performing the operation "Set-Acl" on target "AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local".
VERBOSE: Performing the operation "Set-Acl" on target "AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local".
VERBOSE: Performing the operation "Set-Acl" on target "AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local".
VERBOSE: Performing the operation "Set-Acl" on target "AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local".
VERBOSE: Performing the operation "Set-Acl" on target "AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local".
VERBOSE: Performing the operation "Set-Acl" on target "AD:\OU=Users,OU=Tier 1,OU=Administration,OU=Corp,DC=test,DC=local".
...
```

# Deploy Users  
**Script:** (Create-Users.ps1)  
Now we get to the last step, which is to deploy the administrative accounts for users to be able to manage active directory.

### Requirements
One input file is needed, user-creation.csv and should maintain the following structure. 

| SAMAccountName | Tier 0 | Tier 1 | Tier 2 |
|:---------------|:-------|:-------|:-------|
| john.doe       | TRUE   | TRUE   | TRUE   |
| jane.doe       | FALSE  | FALSE  | FALSE  |

The CSV and the script can be further extended to also include roles for example, but here we cover the basics. 

> User accounts are automatically placed in the Tier Role group for the respective tier, for example: ROLE_Tier 1 Admin. These roles can be user to enforce settings for all admins for the respective tier. Such as PSO's, Logon isolation, etc...

### Run Script
When the CSV file is ready you can run the script and see output for any errors.

```
PS C:\Temp> C:\Temp\Create-Users.ps1
Created user AT0_john.doe
Created user AT1_john.doe
Created user AT2_john.doe
```

> If are 3 values are FALSE there will be no output for this user.

# Conclusion
These are all the steps to setup a basic tiering structure and role based access model for an AD environment. There are still some steps to set before you can consider the environment production ready. For example:
- Set GPO settings
- Create Authentication policies and silos for administrative accounts
- Setup PAW's or an alternative such as a RDP Session Host

Check the other guides within the Active Directory category to find information regarding these topics.
