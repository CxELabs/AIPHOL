# Azure Information Protection New and Advanced Features
### Microsoft Ready Technical Lab 
## MW-SC-TS317

### Introduction

Estimated time to complete this lab

60 minutes

### Objectives

After completing this lab, you will be able to:

- Configure the Azure Information Protection scanner to discover sensitive data 
- Classify and Protect sensitive data discovered by the AIP Scanner
- Synchronize Sensitivity Labels between AIP and SCC
- Use Azure Log Analytics to display centralized reporting for AIP Usage and Data Risk

### Prerequisites

Before working on this lab, you must have:

- Familiarity using Windows 10
- Familiarity with PowerShell

### Lab machine technology

This lab is designed to be completed on Windows 10 VM with the following characteristics:

- Windows 10 Enterprise / Windows Server 2016
- Office 365 ProPlus
- Azure Information Protection Client (1.45.32.0)

Microsoft 365 E5 Tenant credentials will be provided during the event.  If you want to run through this lab after the event, you may use a tenant created through https://demos.microsoft.com or your own Microsoft 365 Tenant. This Lab Guide will be publicly available after the event at https://aka.ms/AIPHOL2

---

===

# Azure Information Protection
[:arrow_left: Home](#introduction)

## Overview

Azure Information Protection (AIP) is a cloud-based solution that can help organizations to protect sensitive information by classifying and (optionally) encrypting documents and emails on Windows, Mac, and Mobile devices. This is done using an organization defined classification taxonomy made up of labels and sub-labels. These labels may be applied manually by users, or automatically by administrators via defined rules and conditions.

The phases of AIP are shown in the graphic below.  

!IMAGE[Phases.png](\Media\Phases.png)

In this lab, we will guide you through addressing all of these phases using some of the newest features of AIP.  We first will perform Discovery using the AIP scanner. We recommend that all customers do this step as it only requires AIP P1 licensing and can help to show customers the risk they are currently facing so they can properly prioritize their security investments. 

We will also show how to use the AIP Scanner in Enforce mode to take advantage of AIP P2 features like Automatic Conditions to help them Classify, Label, and Protect the discovered information easily.

We will help you understand how to Enable and Publish labels in the Security and Compliance Center so they can be used with Mac, Mobile, ISVs (like Adobe PDF), and other unified clients.

Finally, we will demonstrate how to use the new AIP Dashboards to leverage Azure Log Analytics to display actionable information on Usage, Activity, and Data Risk.

!IMAGE[Two overlaying screenshots of the Azure Information Protection scanner's blade in the Azure portal. This blade provides dashboards that consolidate information for all deployed Azure Information Protection scanners, including health status, scan results, classification and policy settings, and more.](\Media\8324-image001.png)

## Objectives

This lab assumes that you are familiar with label and policy creation and that you have seen the operation of conditions in Office applications as these will not be demonstrated.  This lab will use the predefined labels and global policy populated in the demo tenants.

---

===
# Lab Environment Configuration
[:arrow_left: Home](#introduction)

There are a few prerequisites that need to be set up to complete all the sections in this lab.  This Exercise will walk you through the items below.

- [Azure AD User Configuration](#azure-ad-user-configuration)

- [Redeem Azure Pass](#redeem-azure-pass)

- [Configuring Azure Log Analytics](#configuring-azure-log-analytics)

---
# Azure AD User Configuration
[:arrow_up: Top](#lab-environment-configuration)

In this task, we will create new Azure AD users and assign licenses via PowerShell.  In a procduction evironment this would be done using Azure AD Connect or a similar tool to maintain a single source of authority, but for lab purposes we are doing it via script to reduce setup time.

1. [] Log into @lab.VirtualMachine(Scanner01).SelectLink using the password +++Somepass1+++
2. [] Open an **Administrative PowerShell Prompt** and run ```C:\Scripts\AADConfig.ps1```.

1. [] When prompted for the **Tenant name**, **click in the text box** and enter ```@lab.CloudCredential(139).TenantName```.
1. [] When prompted, provide the credentials below:

	```@lab.CloudCredential(139).Username```

	```@lab.CloudCredential(139).Password``` 

	> [!WARNING] If you see any errors, please rerun the script to ensure that licensing is assigned.

	> [!KNOWLEDGE] We are running the PowerShell code below to create the accounts and groups in AAD and assign licenses for EMS E5 and Office E5. This script is also available at [https://aka.ms/labscripts](https://aka.ms/labscripts) as AADConfig.ps1.
    > 
    > #### Azure AD User and Group Configuration
    > $tenantfqdn = "@lab.CloudCredential(139).TenantName"
    > $tenant = $tenantfqdn.Split('.')[0]
	> 
    > #### Build Licensing SKUs
    > $office = $tenant+":ENTERPRISEPREMIUM"
    > $ems = $tenant+":EMSPREMIUM"
	> 
    > #### Connect to MSOLService for licensing Operations
    > Connect-MSOLService -Credential $cred
	> 
    > #### Remove existing licenses to ensure enough licenses exist for our users
    > $LicensedUsers = Get-MsolUser -All  | where {$_.isLicensed -eq $true}
    > $LicensedUsers | foreach {Set-MsolUserLicense -UserPrincipalName $_.UserPrincipalName -RemoveLicenses $office, $ems}
	> 
    > #### Connect to Azure AD using stored credentials to create users
    > Connect-AzureAD -Credential $cred
	> 
    > #### Import Users from local csv file
    > $users = Import-csv C:\users.csv
	> 
    > foreach ($user in $users){
    > 	
    > #### Store UPN created from csv and tenant
    > $upn = $user.username+"@"+$tenantfqdn
	> 
    > #### Create password profile preventing automatic password change and storing password from csv
    > $PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile 
    > $PasswordProfile.ForceChangePasswordNextLogin = $false 
    > $PasswordProfile.Password = $user.password
	> 
    > #### Create new Azure AD user
    > New-AzureADUser -AccountEnabled $True -DisplayName $user.displayname -PasswordProfile $PasswordProfile -MailNickName $user.username -UserPrincipalName $upn
    > }
    > 
    > Start-Sleep -s 10
	> foreach ($user in $users){
	> 
    > #### Store UPN created from csv and tenant
    > $upn = $user.username+"@"+$tenantfqdn
	> 
    > #### Assign Office and EMS licenses to users
    > Set-MsolUser -UserPrincipalName $upn -UsageLocation US
    > Set-MsolUserLicense -UserPrincipalName $upn -AddLicenses $office, $ems
    > }
	> 
    > #### Assign Office and EMS licenses to Admin user
    > $upn = "admin@"+$tenantfqdn
    > Set-MsolUser -UserPrincipalName $upn -UsageLocation US
    > Set-MsolUserLicense -UserPrincipalName $upn -AddLicenses $office, $ems

---

## Redeem Azure Pass
[:arrow_up: Top](#lab-environment-configuration)

For several of the exercises in this lab series, you will require an active subscription.  We are providing an Azure Pass for this purpose.  You will be provided with an Azure Pass code to use with the instructions below.

### Redeeming a Microsoft Azure Pass Promo Code:

1. [] Log into @lab.VirtualMachine(Client01).SelectLink using the password +++Pa$$w0rd+++
2. [] Right-click on **Edge** in the taskbar and click on **New InPrivate window**.

3. [] In the InPrivate window, navigate to ```https://www.microsoftazurepass.com```

4. [] Click the **Start** button to get started.

	> !IMAGE[wdir7lb3.jpg](\Media\wdir7lb3.jpg)
1. [] Log in using the credentials below.

	```@lab.CloudCredential(139).Username```

	```@lab.CloudCredential(139).Password``` 

1. [] Click **Confirm** if the correct email address is listed.

	> !IMAGE[teyx280d.jpg](\Media\teyx280d.jpg)
7. [] Click in the Promo code box and type ```@lab.CloudCredential(221).PromoCode```, then click the **Claim Promo Code** button.

	> !IMAGE[e1l35ko2.jpg](\Media\e1l35ko2.jpg)

	>[!NOTE] It may take up to 5 minutes to process the redemption.

8. [] Scroll to the bottom of the page and click **Next**.

    > !IMAGE[ihrjazqi.jpg](\Media\ihrjazqi.jpg)

	>[!NOTE] You can keep the pre-populated information.

9. [] Check the box to agree to the terms and click **Sign up**.

	> !IMAGE[k2a97g8e.jpg](\Media\k2a97g8e.jpg)

	> [!NOTE] It may take a few minutes to process the request.

---
## Configuring Azure Log Analytics 
[:arrow_up: Top](#lab-environment-configuration)

In order to collect log data from Azure Information Protection clients and services, you must first configure the log analytics workspace.

1. [] In the Azure portal, type the word ```info``` into the **search bar** and press **Enter**, then click on **Azure Information Protection**. 

	> !IMAGE[2598c48n.jpg](\Media\2598c48n.jpg)
	
	> [!HINT] If you do not see the search bar at the top of the portal, click on the **Magnifying Glass** icon to expand it.
	>
	> !IMAGE[ny3fd3da.jpg](\Media\ny3fd3da.jpg)

	> [!KNOWLEDGE] You should automatically be logged into the azure portal.  If not, navigate to ```https://portal.azure.com/``` and log in with the credentials below.
	>
	>```@lab.CloudCredential(139).Username``` 
	>
	>```@lab.CloudCredential(139).Password```

1. [] In the Azure Information Protection blade, under **Manage**, click **Configure analytics (preview)**.

1. [] Next, click on **+ Create new workspace**.

	> !IMAGE[qu68gqfd.jpg](\Media\qu68gqfd.jpg)
1. [] In the Log analytics workspace using the values in the table below and click **OK**.

	|||
	|-----|-----|
	|OMS Workspace|**Type a unique Workspace Name**|
	|Resource Group|```AIP-RG```|
	|Location|**East US**| 

	> [!HINT] The OMS **Workspace name** must be **unique across all of Azure**. The name is not relevant for this lab, so feel free to use random characters.

1. [] Next, back in the Configure analytics (preview) blade, **check the boxes** next to the **workspace** and next to **Enable Content Matches** and click **OK**.

	> !IMAGE[1547437013585](\Media\1547437013585.png)

	> [!KNOWLEDGE] Checking the box next to **Enable Content Matches** allows the **actual matched content** to be stored in the Azure Log Analytics workspace.  This could include many types of sensitive information such as SSN, Credit Card Numbers, and Banking Information.  This option is typically used during testing of automatic conditions and not widely used in production settings due to the sensitive nature of the collected data.  If this is used in a production setting, extreme caution should be taken with securing access to this workspace.

1. [] Click **Yes**, in the confirmation dialog.

	> !IMAGE[zgvmm4el.jpg](\Media\zgvmm4el.jpg)

---

===
# Configuring AIP Scanner for Discovery
[:arrow_left: Home](#azure-information-protection)

Even before configuring an AIP classification taxonomy, customers can scan and identify files containing sensitive information based on the built-in sensitive information types included in the Microsoft Classification Engine.  

> !IMAGE[ahwj80dw.jpg](\Media\ahwj80dw.jpg)

Often, this can help drive an appropriate level of urgency and attention to the risk customers face if they delay rolling out AIP classification and protection.  

In this exercise, we will configure an AIP scanner profile in the Azure portal and install the AIP scanner. Initially, we will run the scanner against repositories in discovery mode.  Later in this lab (after configuring labels and conditions), we will revisit the scanner to perform automated classification, labeling, and protection of sensitive documents. This Exercise will walk you through the items below.

- [AIP Scanner Profile Configuration](#aip-scanner-profile-configuration)
- [AIP Scanner Setup](#aip-scanner-setup)

---
## AIP Scanner Profile Configuration
[:arrow_up: Top](#configuring-aip-scanner-for-discovery)

The new AIP scanner preview client (1.45.32.0) and future GA releases will use the Azure portal central management user interface.  You are now able to manage multiple scanners without the need to sign in to the Windows computers running the scanner, set whether the scanner runs in Discovery or Enforcement mode, configure which sensitive information types are discovered and set repository related settings, like file types scanner, default label etc. Configuration from the Azure portal helps your deployments be more centralized, manageable, and scalable.

> !IMAGE[ScannerUI](\Media\ScannerUI.png)

To make the admin’s life easier we created a repository default that can be set one time on the profile level and can be reused for all added repositories. You can still adjust settings for each repository in case you have a repository that requires some special treatment. 

The AIP scanner operational UI helps you run your operations remotely using a few simple clicks.  Now you can:

- Monitor the status of all scanner nodes in the organization in a single place
- Get scanner version and scanning statistics
- Initiate on-demand incremental scans or run full rescans without having to sign in to the computers running the scanners

> !IMAGE[ScannerUI2](\Media\ScannerUI2.png)

In this task, we will configure the repository default and add a new profile with the repositories we want to scan.

1. [] On @lab.VirtualMachine(Client01).SelectLink, in the Azure Information Protection blade, under **Scanner**, click **Profiles (Preview)**.

	> !IMAGE[ScannerProfiles](\Media\ScannerProfiles.png)

	> [!NOTE] If the Azure portal is not already open, navigate to ```https://aka.ms/ScannerProfiles``` and log in with the credentials below.
	>
	> ```@lab.CloudCredential(139).Username```
	>
	> ```@lab.CloudCredential(139).Password```

1. [] In the Scanner Profiles blade, click the **+ Add** button.

1. [] In the Add a new profile blade, enter ```East US``` for the **Proflie name**.

	> [!Note] The default **Schedule** is set to **Manual**, and **Info types to be discovered** is set to **All**.

1. [] Under **Policy Enforcement**, set the **Enforce** switch to **Off**.

1. [] Note the various additional settings, but **do not modify them**. Click **Save** to complete initial configuration.

	> [!KNOWLEDGE] For additional information on the options available for the AIP scanner profile, see the documentation at [https://aka.ms/ProfileConfiguration](https://aka.ms/ProfileConfiguration)

1. [] Once the save is complete, click on **Configure repositories**.

	> !IMAGE[Configure Repository](\Media\ConfigRepo.png)

1. [] In the Repositories blade, click the **+ Add** button.

1. [] In the Repository blade, under **Path**, type ```\\Scanner01\documents```.

1. [] Under Policy enforcement, make the modifications shown in the table below.

	|Policy|Value|
	|-----|-----|
	|**Default label**|**Custom**|
	||**Confidential \ All Employees**|
	|**Default owner**|**Custom**|
	||```@lab.CloudCredential(139).UserName```|

	> !IMAGE[Repo](\Media\Repo.png)

	> [!KNOWLEDGE] These Policy enforcement settings will set a custom default label of **Confidential \ All Employees** for all files that do not match a policy in this repository.  It will also set the default owner for all files protected by the Scanner to ```@lab.CloudCredential(139).UserName```.

1. [] Click **Save**.

1. [] In the Repositories blade, click the **+ Add** button.

1. [] In the Repository blade, under **Path**, type ```C:\PII```.

1. [] Under Policy enforcement, make the modifications shown in the table below.

	|Policy|Value|
	|-----|-----|
	|**Label files based on content**|**Off**|
	|**Default label**|**Custom**|
	||**Highly Confidential \ All Employees**|
	|**Relabel files**|**On**|

	> !IMAGE[Repo2](\Media\Repo2.png)

	> [!KNOWLEDGE] These Policy enforcement settings will cause all files in the repository to have the same label (**Highly Confidential \ All Employees**).  Additionally, if a file with a different label is added to this repository, the scanner will relabel the label to **Highly Confidential \ All Employees**.

1. [] Click **Save**.

1. [] In the Repositories blade, click the **+ Add** button.

1. [] In the Repository blade, under **Path**, type ```http://Scanner01/documents```.

1. [] Leave all policies at Profile default, and click **Save**.

> [!NOTE] We have now configured all three supported AIP Scanner repository types (**CIFS File Share**, **Local Directory**, and on-premises **SharePoint Document Library**).

---
## AIP Scanner Setup
[:arrow_up: Top](#configuring-aip-scanner-for-discovery)

In this task we will use a script to install the AIP scanner service and create the Azure AD Authentication Token necessary for authentication.

### Installing the AIP Scanner Service

The first step in configuring the AIP Scanner is to install the service and connect the database.  This is done with the Install-AIPScanner cmdlet that is provided by the AIP Client software.  The AIPScanner service account has been pre-staged in Active Directory for convenience.

1. [] Switch to @lab.VirtualMachine(Scanner01).SelectLink and log in using the Credentials below.

	> +++AIP Scanner+++
	>
	> +++Somepass1+++

1. [] Open an **Administrative PowerShell Window** and type ```C:\Scripts\Install-ScannerPreview.ps1``` and press **Enter**. 

1. [] When prompted, enter the Global Admin credentials below:

	> ```@lab.CloudCredential(139).Username```
	>
	> ```@lab.CloudCredential(139).Password```

1. [] In the popup box, click **OK** to accept the default Profile value **East US**.

	> [!NOTE] This script installs the AIP scanner Service using the **local domain user** account (Contoso\\AIPScanner) provisioned for the AIP Scanner. SQL Server is installed locally and the default instance will be used. The script will prompt for **Tenant Global Admin** credentials, the **AIP scanner Profile name**, and finally the AIP Scanner cloud account.  In a production environment, this will likely be the synced on-prem account, but for this demo we created a cloud only account during AAD Configuration earlier in the lab.
	>
	> This script only works if logged on locally to the server as the AIP scanner Service Account, and the service account is a local administrator.  Please see the scripts at https://aka.ms/ScannerBlog for aadditional instructions.

	> [!KNOWLEDGE]  This script will run the code below. This script is available online as Install-ScannerPreview.ps1 at https://aka.ms/labscripts
	> 
	> Add-Type -AssemblyName Microsoft.VisualBasic
	> 
	> $daU = "contoso\AIPScanner"
	> $daP = "Somepass1" | ConvertTo-SecureString -AsPlainText -Force
	> $dacred = New-Object System.Management.Automation.PSCredential -ArgumentList $daU, $daP
	> 	
	> $gacred = get-credential -Message "Enter Global Admin Credentials"
	> 	
	> Connect-AzureAD -Credential $gacred
	> 	
	> $SQL = "Scanner01"
	> 	
	> $ScProfile = [Microsoft.VisualBasic.Interaction]::InputBox('Enter the name of your configured AIP Scanner Profile', 'AIP Scanner Profile', "East US")
	> 	
	> Install-AIPScanner -ServiceUserCredentials $dacred -SqlServerInstance $SQL -Profile $ScProfile
	> 	
	> $Date = Get-Date -UFormat %m%d%H%M
	> $DisplayName = "AIPOBO" + $Date
	> $CKI = "AIPClient" + $Date
	> 	
	> New-AzureADApplication -DisplayName $DisplayName -ReplyUrls http://localhost
	> $WebApp = Get-AzureADApplication -Filter "DisplayName eq $DisplayName"
	> New-AzureADServicePrincipal -AppId $WebApp.AppId
	> $WebAppKey = New-Guid
	> $Date = Get-Date
	> New-AzureADApplicationPasswordCredential -ObjectId $WebApp.ObjectID -startDate $Date -endDate $Date.AddYears(1) -Value $WebAppKey.Guid -CustomKeyIdentifier $CKI
	> 	
	> $AIPServicePrincipal = Get-AzureADServicePrincipal -All $true | Where-Object { $_.DisplayName -eq $DisplayName }
	> $AIPPermissions = $AIPServicePrincipal | Select-Object -expand Oauth2Permissions
	> $Scope = New-Object -TypeName "Microsoft.Open.AzureAD.Model.ResourceAccess" -ArgumentList $AIPPermissions.Id, "Scope"
	> $Access = New-Object -TypeName "Microsoft.Open.AzureAD.Model.RequiredResourceAccess"
	> $Access.ResourceAppId = $WebApp.AppId
	> $Access.ResourceAccess = $Scope
	> 	
	> New-AzureADApplication -DisplayName $CKI -ReplyURLs http://localhost -RequiredResourceAccess $Access -PublicClient $true
	> $NativeApp = Get-AzureADApplication -Filter "DisplayName eq $CKI"
	> New-AzureADServicePrincipal -AppId $NativeApp.AppId
	> 	
	> Set-AIPAuthentication -WebAppID $WebApp.AppId -WebAppKey $WebAppKey.Guid -NativeAppID $NativeApp.AppId
	>
	> Restart-Service AIPScanner
	> Start-AIPScan

1. [] When prompted, enter the AIP Scanner cloud credentials below:

	> ```AIPScanner@@lab.CloudCredential(139).TenantName```
	>
	> ```Somepass1```

1. [] In the Permissions requested window, click **Accept**.

    > !IMAGE[nucv27wb.jpg](\Media\nucv27wb.jpg)

	> [!NOTE] If you get any errors, copy the command from C:\scripts\Set-AIPAuthentication.txt and run it in the Admin PowerShell prompt.
	> Next run the commands below to start the discovery scan
	>
	> ```Restart-Service AIPScanner```
	>
	> ```Start-AIPScan```

	> [!NOTE] An AIP scanner Discovery scan will start directly after aquiring the application access token.

1. [] Wait a minute or so after the script completes until you see a **Visual Studio Just-In-Time Debugger** dialog with a .NET exception.  Press **OK** in the dialog.

	> [!ALERT] This is due to SharePoint startup in the VM environment. This event **must be acknowledged** to complete the discovery scan.

---

===
# Defining Automatic Conditions 
[:arrow_left: Home](#azure-information-protection)

One of the most powerful features of Azure Information Protection is the ability to guide your users in making sound decisions around safeguarding sensitive data.  This can be achieved in many ways through user education or reactive events such as blocking emails containing sensitive data. 

However, helping your users to properly classify and protect sensitive data at the time of creation is a more organic user experience that will achieve better results long term.  In this task, we will define some basic recommended and automatic conditions that will trigger based on certain types of sensitive data.

1. [] Switch to @lab.VirtualMachine(Client01).SelectLink and log in with the password +++@lab.VirtualMachine(Client01).Password+++.

1. [] In the **AIP blade**, under **Analytics** on the left, click on **Data discovery (Preview)** to view the results of the discovery scan we performed previously.

	> !IMAGE[Dashboard.png](\Media\Dashboard.png)

	> [!KNOWLEDGE] The screenshot above shows a discovery only scan. Notice that there are no labeled or protected files shown at this time.  This uses the AIP P1 discovery functionality available with the AIP Scanner. Only the predefined Office 365 Sensitive Information Types are available with AIP P1 as Custom Sensitive Information Types require automatic conditions to be defined, which is an AIP P2 feature.

	> [!ALERT] It is very likely that the dashboard in your lab will not be populated at this point as you have just started the discovery scan. Continue with the lab and we will come back to see the results later.

1. [] Under **Classifications** on the left, click **Labels** then expand **Confidential**, and click on **All Employees**.

	^IMAGE[Open Screenshot](\Media\jyw5vrit.jpg)
1. [] In the Label: All Employees blade, scroll down to the **Configure conditions for automatically applying this label** section, and click on **+ Add a new condition**.

	> !IMAGE[cws1ptfd.jpg](\Media\cws1ptfd.jpg)
1. [] In the Condition blade, in the **Select information types** search box, type ```EU``` and check the boxes next to the **items shown below**.

	> !IMAGE[xaj5hupc.jpg](\Media\xaj5hupc.jpg)

1. [] Click **Save** in the Condition blade and **OK** to the Save settings prompt.

	^IMAGE[Open Screenshot](\Media\41o5ql2y.jpg)
1. [] In the Labels: All Employees blade, in the **Configure conditions for automatically applying this label** section, click **Automatic**.

1. [] Click **Save** in the Label: All Employees blade and **OK** to the Save settings prompt.

	^IMAGE[Open Screenshot](\Media\rimezmh1.jpg)
1. [] Press the **X** in the upper right-hand corner to close the Label: All Employees blade.

	^IMAGE[Open Screenshot](\Media\em124f66.jpg)
1. [] Next, expand **Highly Confidential** and click on the **All Employees** sub-label.

	^IMAGE[Open Screenshot](\Media\2eh6ifj5.jpg)
1. [] In the Label: All Employees blade, scroll down to the **Configure conditions for automatically applying this label** section, and click on **+ Add a new condition**.

	^IMAGE[Open Screenshot](\Media\8cdmltcj.jpg)
1. [] In the Condition blade, in the search bar type ```credit``` and check the box next to **Credit Card Number**.

	^IMAGE[Open Screenshot](\Media\9rozp61b.jpg)
1. [] Click **Save** in the Condition blade and **OK** to the Save settings prompt.

	^IMAGE[Open Screenshot](\Media\ie6g5kta.jpg)
1. [] In the Labels: All Employees blade, in the **Configure conditions for automatically applying this label** section, click **Automatic**.

	> [!HINT] The policy tip is automatically updated when you switch the condition to Automatic.
	>
	> !IMAGE[245lpjvk.jpg](\Media\245lpjvk.jpg)

1. [] Click **Save** in the Label: All Employees blade and **OK** to the Save settings prompt.

	^IMAGE[Open Screenshot](\Media\gek63ks8.jpg)

1. [] Press the **X** in the upper right-hand corner to close the Label: All Employees blade.

	^IMAGE[Open Screenshot](\Media\wzwfc1l4.jpg)

---

===
# Security and Compliance Center 
[:arrow_left: Home](#azure-information-protection)

In this exercise, we will migrate your AIP Labels and activate them in the Security and Compliance Center.  This will allow you to see the labels in Microsoft Information Protection based clients such as Office 365 for Mac and Mobile Devices.

Although we will not be demonstrating these capabilities in this lab, you can use the tenant information provided to test on your own devices.

---
## Activating Unified Labeling
[:arrow_up: Top](#security-and-compliance-center)

In this task, we will activate the labels from the Azure Portal for use in the Security and Compliance Center.

1. [] On @lab.VirtualMachine(Client01).SelectLink, in the AIP blade, click on **Unified labeling (Preview)**.

	> !IMAGE[Unified Labeling](\Media\Unified.png)

3. [] Click **Activate** and **Yes**.

	> !IMAGE[o0ahpimw.jpg](\Media\o0ahpimw.jpg)

	>[!NOTE] You should see a message similar to the one below.
	>
	> !IMAGE[SCCMigration.png](\Media\SCCMigration.png) 

1. [] In a new tab, browse to ```https://protection.office.com/``` and click on **Classifications** and **Labels** to review the migrated labels. 

	>[!NOTE] Keep in mind that now the SCC Sensitivity Labels have been activated, so any modifications, additions, or deletions will be syncronised to Azure Information Protection in the Azure Portal. There are some functional differences between the two sections (DLP in SCC, HYOK & Custom Permissions in AIP), so please be aware of this when modifying policies to ensure a consistent experience on clients. 

---
## Deploying Policy in SCC
[:arrow_up: Top](#security-and-compliance-center)

The previous step enabled the AIP labels for use in the Security and Compliance Center.  However, this did not also recreate the policies from the AIP portal. In this step we will publish a Global policy like the one we used in the AIP portal for use with unified clients.

1. [] In the Security and Compliance Center, under Classifications, click on **Label policies**.

2. [] In the Label policies pane, click **Publish labels**.

	^IMAGE[Open Screenshot](\Media\SCC01.png)
3. [] On the Choose labels to publish page, click the **Choose labels to publish** link.

	^IMAGE[Open Screenshot](\Media\SCC02.png)
4. [] In the Choose labels pane, click the **+ Add** button.

	^IMAGE[Open Screenshot](\Media\SCC03.png)
5. [] Click the box next to **Display name** to **select all labels**, then click the **Add** button.

	^IMAGE[Open Screenshot](\Media\SCC04.png)
6. [] Click the **Done** button.

	^IMAGE[Open Screenshot](\Media\SCC05.png)
7. [] Back on the Choose labels to publish page, click the **Next** button.

	^IMAGE[Open Screenshot](\Media\SCC06.png)
8. [] On the Publish to users and groups page, notice that **All users** are included by default. If you were creating a scoped policy, you would choose specific users or groups to publish to. Click **Next**.

	^IMAGE[Open Screenshot](\Media\SCC07.png)
9. [] On the Policy settings page, select the **General** label from the drop-down next to **Apply this label by default to documents and email**.
10. [] Check the box next to **Users must provide justification to remove a label or lower classification label** and click the **Next** button.

	> !IMAGE[Open Screenshot](\Media\SCC08.png)

11. [] In the Name textbox, type ```Global Policy``` and for the Description type ```This is the default global policy for all users.``` and click the **Next** button.

	^IMAGE[Open Screenshot](\Media\SCC09.png)
12. [] Finally, on the Review your settings page, click the **Publish** button.

	> !IMAGE[Open Screenshot](\Media\SCC10.png)

---

===
# Classification, Labeling, and Protection with the Azure Information Protection Scanner 
[:arrow_left: Home](#azure-information-protection)

The Azure Information Protection scanner allows you to  classify and protect sensitive information stored in on-premises CIFS file shares and SharePoint sites using an automated process.  This is ideal for protecting large datastores with sensitive information that is not centrally located.  Most customers will prefer this option over moving and manually protecting sensitive documents.  

In this exercise, we will run the AIP Scanner in enforce mode to classify and protect the identified sensitive data. This Exercise will walk you through the items below.

- [Enforcing Configured Rules](#enforcing-configured-rules)
- [Reviewing Protected Documents](#reviewing-protected-documents)
- [Reviewing the Dashboards](#reviewing-the-dashboards)

---

## Enforcing Configured Rules 
[:arrow_up: Top](#classification-labeling-and-protection-with-the-azure-information-protection-scanner)

In this task, we will modify the AIP scanner Profile to enforce the conditions we set up and have it run on all files using the Start-AIPScan command.

1. [] On @lab.VirtualMachine(Client01).SelectLink, return to **Scanner > Profiles (Preview)** in the Azure Portal.

	> [!NOTE] If needed, navigate to ```https://aka.ms/ScannerProfiles``` and log in with the credentials below:
	>
	> ```@lab.CloudCredential(139).Username```
	>
	> ```@lab.CloudCredential(139).Password```

2. [] Click on the **East US** profile.

1. [] In the East US profile, under Profile settings, configure the settings in the table below.

	|**Policy**|**Value**|
	|-----|-----|
	|**Schedule**|**Always**|
	|**Info types to be discovered**|**Policy only**|
	|**Enforce**|**On**|
	
	> !IMAGE[Enforce](\media\Enforce.png)

	> [!NOTE] These settings will cause the scanner to run continuously on the repositories, make the scanner only look for the sensitive information types we defined in conditions, and Enforce the labeling and protection of files based on those conditions. Leave all other settings in their current state.

1. [] Click **Save** then click the **X** to close the blade.

1. [] Next, under Scanner, click on **Nodes**.

	> !IMAGE[Nodes](\Media\Nodes.png)

1. [] Highlight the row containing **Scanner01.Contoso.Azure**, and click **Scan now** in the command list above.

	> !IMAGE[ScanNow](\Media\ScanNow.png)

1. [] The previous command can take up to 5 minutes to run on the AIP scanner Server. Follow the commands below to accelerate the process.

	1. [] Switch to @lab.VirtualMachine(Scanner01).SelectLink and log in with the password +++@lab.VirtualMachine(Scanner01).Password+++.

	1. [] In an Administrative PowerShell window, run the ```Start-AIPScan``` command.

---

## Reviewing Protected Documents 
[:arrow_up: Top](#classification-labeling-and-protection-with-the-azure-information-protection-scanner)

Now that we have Classified and Protected documents using the scanner, we can review the documents to see their change in status.

1. [] Switch to @lab.VirtualMachine(Client01).SelectLink and log in with the password +++@lab.VirtualMachine(Client01).Password+++.

2. [] Navigate to ```\\Scanner01.contoso.azure\documents```. 

	> If needed, use the credentials below:
	>
	>```Contoso\LabUser```
	>
	>```Pa$$w0rd```

	^IMAGE[Open Screenshot](\Media\hipavcx6.jpg)
3. [] Open one of the **Contoso Purchasing Permissions** documents.
1. [] When prompted, provide the credentials below:

	> ```EvanG@@lab.CloudCredential(139).TenantName```
	>
	> ```pass@word1```

1. [] Click **Yes** to allow the organization to manage the device.
	
	> [!NOTE] Observe that the document is classified as Highly Confidential \ All Employees. 
    >
    > !IMAGE[s1okfpwu.jpg](\Media\HCAE.jpg)

4. [] Next, in the same documents folder, open one of the **pdf files**.
5. [] When prompted by Adobe, enter ```EvanG@@lab.CloudCredential(139).TenantName``` and press **Next**.
6. [] Check the box to save credentials and press **Yes**.
1. [] Click **Accept** in the **Permissions requested** dialog.

	> [!NOTE] The PDF will now open and **display the sensitivity** across the top of the document.
	>
	> !IMAGE[PDF](\Media\PDF.png)

	> [!Knowledge] The latest version of Acrobat Reader DC and the MIP Plugin have been installed on this system prior to the lab. Additionally, the sensitivity does not display by default in Adobe Acrobat Reader DC.  You must make the modifications below to the registry to make this bar display.
	>
	> In **HKEY_CURRENT_USER\Software\Adobe\Acrobat Reader\DC\MicrosoftAIP**, create a new **DWORD** value of **bShowDMB** and set the **Value** to **1**.
	>
	> !IMAGE[1547416250228](\Media\1547416250228.png)

---
## Reviewing the Dashboards 
[:arrow_up: Top](#classification-labeling-and-protection-with-the-azure-information-protection-scanner)

We can now go back and look at the dashboards and observe how they have changed.

1. [] On @lab.VirtualMachine(Client01).SelectLink, open the browser that is logged into the Azure Portal.

	> [!ALERT] Some of the content shown in this dashboard will not be present because we did not perform manual labeling.  This content has been left in to show the capabilities of the reports.

1. [] Under **Analytics**, click on **Usage report (Preview)**.

	> [!NOTE] Observe that there are now entries from the AIP scanner, File Explorer, Microsoft Outlook, and Microsoft Word based on our activities in this lab. 
	>
	> !IMAGE[Usage.png](\Media\newusage.png)

	> [!KNOWLEDGE] If there is no data in the dashboard, open a **File Explorer** window, and browse to ```\\Scanner01.contoso.azure\c$\users\aipscanner\AppData\Local\Microsoft\MSIP\Scanner\Reports```.
	> Review the summary txt and detailed csv files available there. You may need to open the largest zipped files to find the scan that actually labeled the content.
	>
	> The details contained in the DetailedReport.csv shows the types of sensitive data found by the scanner. This data will be shown in AIP analytics after some processing time.
	>
	> !IMAGE[9y52ab7u.jpg](\Media\9y52ab7u.jpg)

2. [] Next, under Analytics, click on **Activity logs (preview)**.
   
    > [!NOTE] We can now see activity from various users and clients including the AIP Scanner and specific users. 
	>
	> !IMAGE[activity.png](\Media\activity.png)
	
1. [] Select the drop-down list under **Labels** and check the box next to **Highly Confidential \ All Employees**. 

	> !IMAGE[activity2.png](\Media\activity2.png)

1. [] Click on one of the entries to bring up the **Activity Details** panel.

	> [!NOTE] In the Activity Details panel, you can see all of the details related to the classification, labeling, and protection of the file. The level of detail shown below is only available if you checked the box to Enable document content matches under Configure analytics (Preview). 
	>
	> !IMAGE[activity2.png](\Media\activity3.png)	

3. [] Finally, click on **Data discovery (Preview)**.

	> [!NOTE] In the Data discovery dashboard, you can see a breakdown of how files are being protected and locations that have sensitive content.
	>
	> !IMAGE[Discovery.png](\Media\Discovery2.png)
	> 
	> If you click on one of the locations, you can drill down and see the content that has been protected on that specific device or repository.
	>
	> !IMAGE[discovery2.png](\Media\discovery2b.png)
	

===
# AIP Lab Complete 
[:arrow_left: Home](#azure-information-protection)

Congratulations! You have completed the Azure Information Protection Hands on Lab. 

In this lab, you have successfully completed the exercises below covering the four phases of the Information Protection Lifecycle.  We first Discovered and Classified information using the AIP scanner. We showed how to enable unified labeling for use with Mac, Mobile, and 3rd party applications like Adobe Acrobat.  We then Labeled and Protected documents with the AIP scanner in Enforce mode. Finally, we reviewed the Monitoring using Azure Log Analytics and the new AIP Dashboards.

- Sensitive data Discovery using the AIP scanner and the new cloud UI
- Enabling Sensitivity Labels in the Microsoft 365 Security and Compliance Center
- Publishing a Label Policy in the Microsoft 365 Security and Compliance Center
- Automatic Classification and Protection of sensitive data using the AIP scanner
- Review of protected Office and Adobe PDF documents
- Review of the new Azure Log Analytics Dashboards

We hope you enjoyed this lab! Please fill out the survey and let us know what was valuable and what was not so that we may improve the experience for future labs. Thanks!

> !IMAGE[cat](\Media\ninjacat.png)

!INSTRUCTIONS[https://blogs.msdn.microsoft.com/oldnewthing/20160804-00/?p=94025][ninja-cat]
[https://blogs.msdn.microsoft.com/oldnewthing/20160804-00/?p=94025](https://blogs.msdn.microsoft.com/oldnewthing/20160804-00/?p=94025)

