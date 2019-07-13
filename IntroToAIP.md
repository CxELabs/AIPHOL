# Introduction to Azure Information Protection, Unified Labeling, and Office 365 Message Encryption

Azure Information Protection (AIP) is a cloud-based solution that can help organizations to protect sensitive information by classifying and (optionally) encrypting documents and emails on Windows, Mac, and Mobile devices. This is done using an organization defined classification taxonomy made up of labels and sub-labels. These labels may be applied manually by users, or automatically by administrators via defined rules and conditions.

The phases of AIP are shown in the graphic below.  

!IMAGE[Phases.png](Media/Phases.png)

This lab will cover the **Classify and label** and **Protect and control access** phases.

---

## Objectives

In this lab, you will:

- Learn how to use AIP to classify, label, protect, and control access to sensitive data
- Learn basics about Azure Information Protection including configuring labels and policy within the Azure portal
- Learn about Unified Labeling and how to configure and deploy labels within the Security and Compliance Center for use with Windows, Mac, Mobile, and 3rd party Unified Labeling clients
- Learn the difference between the AIP client (classic) and the AIP Unified Labeling client and when each should be deployed
- Learn how to configure recommended and automatic conditions in the Azure portal and the SCC
- Test manual, recommended, and automatic labeling using the AIP client on Windows systems
- Learn about sending email messages with Office 365 Message Encryption for secure collaboration to any recipient

---
Estimated time to complete this lab

60 minutes

---
===
## Where to Begin
With General Availability of Unified Labeling clients and Sensitivity Labels in the Security and Compliance Center, there has been some confusion on where customers should start their deployment and which clients they should use. This is a common question, and one that requires understanding of the capabilities available in Azure Information Protection versus what is currently available in the Azure Information Protection Unified Labeling client. Before we get into these, we need to introduce what each of these are and where the policies they use are configured.

## Azure Information Protection client 
The Azure Information Protection client (classic) has been available since Azure Information Protection was first announced as a new service for classifying and protecting files and emails. This client downloads labels and policy settings from Azure, and you configure the Azure Information Protection policy from the Azure portal. This client has a version nember of 1.xx and is installed using **AzInfoProtection.exe** available at ```https://aka.ms/AIPclient```.

## Azure Information Protection Unified Labeling client 
The Azure Information Protection Unified Labeling client is a more recent addition, to support the unified labeling store that multiple applications and services support. This client downloads sensitivity labels and policy settings from the following admin centers: The Office 365 Security & Compliance Center, the Microsoft 365 security center, and the Microsoft 365 compliance center. This client has a version nember of 2.xx and is installed using **AzInfoProtection_ul.exe** available at ```https://aka.ms/AIPclient```.

## Choosing which client to use

Install the Azure Information Protection unified labeling client for labels that can also be used by MacOS, iOS, and Android, and if you don’t need advanced features, such as advanced client settings, user-defined permissions, an on-premises key (HYOK), or the scanner for on-premises data stores.

Install the Azure Information Protection client (classic) if you need advanced features that are not yet available in the unified labeling client, but the labels can't be used on other client platforms.
Currently, the Azure Information Protection client (classic) and the Azure Information Protection unified labeling client don't have parity for their features. However, expect this gap to close and then, new features to be added only to the Azure Information Protection unified labeling client. For this reason, we recommend you deploy the Azure Information Protection unified labeling client if its current feature set and functionality meet your business requirements. If not, or if you have configured labels in the Azure portal that you haven't yet migrated to the unified labeling store , use the Azure Information Protection client (classic).

You can install both clients in the same environment to support different business requirements depending on which advanced features are necessary. For this, we recommend you migrate the labels in the Azure portal so that both sets of clients share the same set of labels for ease of administration.

In this lab, we will focus on the Azure Information Protection client (classic) which is configured in the Azure Portal and discuss the migration of labels to the SCC so they can also be used with UL clients.

## Logging into the Azure Portal

To start learning about Azure Information Protection, you first need to log into the AIP blade in the Azure Portal.  The instructions below show a quick way to accomplish this.

1. [] On Client01, log in with the credentials below

	>**LisaV**
	>
	>**HighImpactUser1!**
1. [] From the taskbar at the bottom of the screen, launch **Internet Explorer**

1. [] **Maximize** the browser window

1. [] In the **Internet Explorer** address bar, type **```https://aka.ms/AIPConsole```**

1. [] In the **Email or phone** field, enter **@lab.CloudCredential(139).Username** and click **Next**

1. [] In the **Password** field, enter **@lab.CloudCredential(139).Password** and click **Sign in**

1. [] In the **Stay signed in?** window, click **Yes**

1. [] If you get  a **Welcome to Microsoft Azure** popup window, click **Maybe Later** to skip the tour

## Default Label Configuration

In new AIP tenants, Microsoft is no longer staging labels in the tenants by default.  The demo tenants provided for this lab may be in this new default state with no labels.  In this step, we will walk through the process of generation the default label set and assigning them to the global policy.

> **If the labels already exist in your test tenant, please jump directly to the [User Interface Walkthrough](#user-interface-walkthrough).**

1. [] In the Azure Information Protection blade, under **Classifications** click on **Labels**.
1. [] In the Labels blade, at the top, click the **+ Generate default labels** button.

	>This process may take a few minutes.

	!IMAGE[Generate.png](Media/Generate.png)

	>You will see an image like the one below once the generation is complete.
	
	!IMAGE[Complete.png](Media/Complete.png)

1. [] Next, under Classifications on the left, click **Policies**.
1. [] In the Policies blade, click on **Global** to open the policy.

	!IMAGE[global.png](Media/Global.png)
1. [] In the Policy: Global blade, wait until you see **No labels** under the LABEL DISPLAY NAME column, then click **Add or remove labels**.

	!IMAGE[nolabels.png](Media/nolabels.png)
1. [] In the Policy: Add or remove labels blade, **check the boxes next to each of the labels** and click **OK**.

	!IMAGE[alllabels.png](Media/alllabels.png)
1. [] Back in the Policy: Global blade, click **Save** and **OK**.
1. [] After the policy has successfully saved, click the **X** in the upper right corner to close the Policy:Global blade.
1. [] Next, under **Classifications** click on **Labels**.
1. [] Expand the **Confidential** and **Highly Confidential** parent labels and your labels should look like the image below.

	!IMAGE[defaults.png](Media/defaults.png)

    >Creating default labels in this way also creates them in the Security and Compliance Center so you will not need to Activate Unified Labeling when you reach that step later in this lab.

    !IMAGE[ULActivated.png](Media/ULActivated.png)

---

## User Interface Walkthrough

Now that we are logged in and have our labels available, we will discuss the various elements of the AIP blade.

1. [] In the **General** section, click on **Quick start**.

    >This section contains helpful information around getting started using Azure Information Protection
1. [] In the **Classifications** section, click on **Labels**.

    >Note: We are intentionally skipping over the **Analytics** section and we will return to that at the end of this section.

	>In the **Labels** section, you will see the default labels configured for the AIP demo tenants. 
	>These demo labels are modeled after the labels that Microsoft uses internally and are highly recommended as a baseline for customers that 
	>do not already have an established and effective classification taxonomy. More details on how Microsoft came up with this taxonomy and how 
	>Microsoft deployed AIP are available in a video online at ```https://aka.ms/AIPShowcase```.

	!IMAGE[UI01](Media/UI01.png)

	>In the next section of this lab, we will add and modify labels to show the basic functionality and features of labeling and protection.

1. [] Next, under **Classifications**, click on **Policies**.
1. [] In the **Policies** blade, click on the **Global** policy.

	>In the **Policy: Global** blade, you have settings that will apply to all users within the organization unless superceded by more specific scoped policies.  We will discuss the various settings available in Global and Scoped policies in a later section of this lab.
1. [] At the top of the page, click on **Azure Information Protection - Policies** to return to the **Policies** blade. 
1. [] On the left, under **Scanner**, click **Nodes**.

	>This blade will show any configured AIP scanner nodes reporting in to this tenant.  This allows you to monitor the status of the AIP scanner and see details like version, scan rate, and number of files scanned.  We will not be installing the AIP scanner in this lab but installation and configuration of the AIP scanner is available in other labs in this series.

	!IMAGE[](Media/nodes.png)
1. [] Next, under **Scanner**, click on **Profiles**.

	>This blade is also related to the AIP scanner and is where you configure repositories and global settings for your AIP scanner instances.  We go into more detail on this in the AIP scanner lab that is part of this lab series.

1. [] On the left, under **Manage**, click on **Configure analytics**.

	>This blade is used for configuring AIP analytics.  This is done by connecting an existing Azure Log Analytics workspace or creating a new one using the **+ Create new workspace** link. Azure log analytics workspaces require an Azure subscription so we will not be able to demonstrate this functionality in this lab, but we will provide screenshots below of the various Analytics dashboards.
	>
	>There is an additional option at the bottom of this blade to **Enable deeper analytics into your sensitive data**. This option is usually only used during testing as it allows the clients and scanner to collect the actual sensitive data contained in files rather than simply identifying that the data exists.
1. [] Below we will review some visualizations of the AIP Analytics

	>The **Usage report** can be used to show labeling and protection data for the last 31 days.  It can also show how many users and devices have connected to your tenant. This is useful information for knowing how how active your users are and if your classification taxonomy is effective.
	>
	>You can also see the distribution of labels and which applications they are primarily being used in.  This will display Office applications, AIP scanner, File Explorer (right-click), and any other  Microsoft and 3rd party applications that utilize the MIP SDK.

	!IMAGE[usage](Media/usage.png)

	>The **Activity logs** report shows all AIP activities that have taken place in a specified period of time.  This defaults to 31 days, but can be used to target specific timeframes for investigation.  
	>
	>This report can be filtered and you can click on individual records to see detailed information about the activity.  We do not go deep into the capabilities in this lab but we will cover this more in the Advanced AIP lab.

	!IMAGE[endpointdiscovery](Media/endpointdiscovery.png)

	>The **Data discovery** report shows a visualization of all labels that have been applied over the last 31 days and all the discovered information types in any scanned files.  You can filter on specific types of information or specific locations including file repositories, on-premises sharepoint libraries, and even specific endpoints.
	>
	>When used in conjunction with Microsoft Defender ATP, you can identify risky devices and see what sensitive information they contain to help prioritize remediation efforts.

	!IMAGE[DataDiscovery](Media/DataDiscovery.png)

	>The **Recommendations** report contains details on files that contain sensitive information that are not currently protected using recommended or automatic conditions.  Without recommended or automatic conditions in place for sensitive data types, users may inadvertantly include sensitive information in documents and not label and protect them appropriately.
	>
	>The Recommendations report also shows locations that contain sensitive information that are not currently being monitored by the AIP scanner or Microsoft Cloud Application Security.  This can help admins locate file repositories that branch offices are using to better target them for discovery and protection.

	!IMAGE[recommendations](Media/recommendations.png)
1. [] Next, under **Manage**, click on **Languages**.

	>The languages blade allows you to select languages that you would like to localize.  This is done by checking the boxes next to each language that you will be providing localization for and clicking **Export**.
	>
	>You will get an **exported localizations.zip** file containing xml files for each of the selected languages.  Once you are done, you can zip these up and import them in the same location.
1. [] Next, under **Manage**, click on **Protection activation**.

	>The Protection activation blade shows the current status of Azure RMS protection in the tenant.  This setting is required to be activated to use the protection capabilities of AIP.  Most tenants have this activated by default.
1. [] Finally, Under **Manage**, click **Unified labeling**.

	>Unified labeling was described earlier in this section.  Activation of unified labeling moves the AIP labels to the Security and Compliance Center back-end.  Once activated, any changes that are made in the Azure portal will also be reflected in the Security and Compliance Center as there is only one copy of the labels.
	>
	>The **Publish** button in this blade is needed if you make changes to labels in the Security and Compliance Center and want those changes reflected on endpoints running the AIP client (classic).  We recommend continuing to manage labels from the Azure portal until you have migrated globally to the AIP unified labeling client.

---
## Review

In this section, we have reviewed the various blades contained in the AIP console of the Azure portal.

In the next section, we will demonstrate the creation and modification of labels and policies, and walk through the various settings that are available for each.

---
===
# Configuring Labels and Policies

This section demonstrates using the Azure Information Protection blade in the Azure portal to configure policies and sub-labels.  We will create a new sub-label and configure protection and then modify an existing sub-label.  We will also create a label that will be scoped to a specific group.  

Next, we will configure AIP Global Policy to use the General sub-label as default, and finally, we will configure a scoped policy to use the new scoped label by default for Word, Excel, and PowerPoint while still using General as default for Outlook. 

---
## Creating, Configuring, and Modifying Sub-Labels

In this task, we will configure a label protected for internal audiences that can be used to help secure sensitive data within your company.  By limiting the audience of a specific label to only internal employees, you can dramatically reduce the risk of unintentional disclosure of sensitive data and help reduce the risk of successful data exfiltration by bad actors.  

However, there are times when external collaboration is required, so we will configure a label to match the name and functionality of the Do Not Forward button in Outlook.  This will allow users to more securely share sensitive information outside the company to any recipient.  By using the name Do Not Forward, the functionality will also be familiar to what previous users of AD RMS or Azure RMS may have used in the past.

1. [] In the Azure Information Protection blade, under **Classifications** in the left pane, click on **Labels** to load the Azure Information Protection – Labels blade.

	!IMAGE[](Media/mhocvtih.jpg)

1. [] In the Azure Information Protection – Labels blade, right-click on **Confidential** and click **Add a sub-label**.

	!IMAGE[](Media/uktfuwuk.jpg)

1. [] In the Sub-label blade, type ```Contoso Internal``` for the **Label display name** and for **Description** enter text similar to ```Confidential data that requires protection, which allows Contoso Internal employees full permissions. Data owners can track and revoke content.```

	!IMAGE[](Media/4luorc0u.jpg)

1. [] Then, under **Set permissions for documents and emails containing this label**, click **Protect**, and under **Protection**, click on **Azure (cloud key)**.

	!IMAGE[](Media/tp97a19d.jpg)

1. [] In the Protection blade, click **+ Add Permissions**.

	!IMAGE[](Media/layb2pvo.jpg)

1. [] In the Add permissions blade, click on **+ Add contoso – All members** and click **OK**.

	!IMAGE[](Media/zc0iuoyz.jpg)

1. [] In the Protection blade, click **OK**.

1. [] In the Sub-label blade, scroll down to the **Set visual marking (such as header or footer)** section.
1. [] Under **Documents with this label have a header**, click **On**.

1. [] Use the values in the table below to configure the Header.

	| Setting          | Value            |
	|:-----------------|:-----------------|
	| Header text      | ```Contoso Internal``` |
	| Header font size | ```24```               |
	| Header color     | Purple           |
	| Header alignment | Center           |

	> These are sample values to demonstrate marking possibilities and **NOT** a best practice.

	!IMAGE[](Media/0vdoc6qb.jpg)

1. [] To complete creation of the new sub-label, click the **Save** button and then click **OK** in the Save settings dialog.

1. [] In the Azure Information Protection - Labels blade, expand **Confidential** (if necessary) and then click on **Recipients Only**.

	!IMAGE[](Media/eiiw5zbg.jpg)

1. [] In the Label: Recipients Only blade, change the **Label display name** from **Recipients Only** to ```Do Not Forward```.

	!IMAGE[](Media/v54vd4fq.jpg)

1. [] Next, in the **Set permissions for documents and emails containing this label** section, under **Protection**, click **Azure (cloud key): User defined**.

	!IMAGE[](Media/qwyranz0.jpg)

1. [] In the Protection blade, under **Set user-defined permissions (Preview)**, verify that only the Box next to **In Outlook apply Do Not Forward** is checked, then click **OK**.

	>  Although there is no action added during this step, it is included to show that this label will only display in Outlook and not in Word, Excel, PowerPoint or File Explorer.

1. [] Click **Save** in the Label: Recipients Only blade and **OK** to the Save settings prompt. 

1. []  Click the **X** in the upper right corner of the blade to close.

---

## Configuring Global Policy

In this task, we will assign the new sub-label to the Global policy and configure several global policy settings that will increase Azure Information Protection adoption among your users and reduce ambiguity in the user interface.

1. [] In the Azure Information Protection blade, under **Classifications** on the left, click **Policies**. 
1. [] Click the **Global** policy.

	!IMAGE[](Media/24qjajs5.jpg)

1. [] In the Policy: Global blade, **wait for the labels to load**.

	> The policies should look like the image below.  If they show as loading, refresh the full browser on this page and go back into the **Global** policy and they should load.
	
	!IMAGE[labels.png](Media/labels.png)

1. [] Below the labels, click **Add or remove labels**.

1. [] In the Policy: Add or remove labels blade, ensure that the **Boxes** next to **all labels including the new Contoso Internal label** are **checked** and click **OK**.

1. [] In the Policy: Global blade, under the **Configure settings to display and apply on Information Protection end users** section, configure the policy to match the settings shown in the table and image below.

	| Setting | Value |
	|:--------|:------|
	| Select the default label | General |
	|All documents and emails must have a label…|On|
	|Users must provide justification to set a lower…|On|
	|For email messages with attachments, apply a label…|Automatic|
	|Display the Information Protection Bar in Office apps|On|
	|Add the Do Not Forward button to the Outlook ribbon|Off|

	!IMAGE[](Media/mtqhe3sj.jpg)

1. [] Click **Save**, then **OK** to complete configuration of the Global policy.

1. [] Click the **X** in the upper right corner to close the Policy: Global blade.

---

## Creating a Scoped Label and Policy

Now that you have learned how to work with global labels and policies, we will create a new scoped label and policy for the Legal team at Contoso.  

1. [] Under **Classifications** on the left, click **Labels**.

	!IMAGE[](Media/50joijwb.jpg)

1. [] In the Azure Information Protection – Labels blade, right-click on **Highly-Confidential** and click **Add a sub-label**.

	!IMAGE[](Media/tasz9t0i.jpg)

1. [] In the Sub-label blade, enter ```Legal Only``` for the **Label display name** and for **Description** enter ```Data is classified and protected. Legal department staff can edit, forward and unprotect.```

	!IMAGE[](Media/lpvruk49.jpg)

1. [] Then, under **Set permissions for documents and emails containing this label**, click **Protect** and under **Protection**, click **Azure (cloud key)**.

	!IMAGE[](Media/6ood4jqu.jpg)

1. [] In the Protection blade, under **Protection settings**, click the **+ Add permissions** link.

	!IMAGE[ozzumi7l.jpg](Media/ozzumi7l.jpg)

1. [] In the Add permissions blade, click **+ Browse directory**.

	!IMAGE[](Media/2lvwim24.jpg)

1. [] In the AAD Users and Groups blade, **wait for the names to load**, then check the Boxes next to **Adele Vance** and **Alex Wilber**, and click the **Select** button.

	!IMAGE[](Media/scopedlabel.png) 

	> In a production environment, you will typically use a synced or Azure AD Group rather than choosing individuals.

1. [] In the Add permissions blade, click **OK**.

1. [] In the Protection blade, under **Allow offline access**, reduce the **Number of days the content is available without an Internet connection** value to ```3``` and press **OK** .

	>  This value determines how many days a user will have offline access from the time a document is opened, and an initial Use License is acquired.  While this provides convenience for users, it is recommended that this value be set appropriately based on the sensitivity of the content.

	!IMAGE[](Media/j8masv1q.jpg)

1. [] Click **Save** in the Sub-label blade and **OK** to the Save settings prompt to complete the creation of the Legal Only sub-label.

1. [] In the Azure Information Protection blade, under **Classifications** on the left, click **Policies** then click the **+Add a new policy** link.

	!IMAGE[](Media/ospsddz6.jpg)

1. [] In the Policy blade, for Policy name, type ```No Default Label Scoped Policy``` and click on **Select which users or groups get this policy. Groups must be email-enabled.**

	!IMAGE[1sjw3mc7.jpg](Media/1sjw3mc7.jpg)

1. [] In the AAD Users and Groups blade, click on **Users/Groups**.  
1. [] Then in the second AAD Users and Groups blade, **wait for the names to load** and check the Boxes next to  **Adele Vance** and **Alex Wilber**.

1. [] Click the **Select** button.
1. [] Finally, click **OK**.

	!IMAGE[](Media/onne7won.jpg)

1. [] In the Policy blade, under the labels, click on **Add or remove labels** to add the scoped label.

	!IMAGE[b6e9nbui.jpg](Media/b6e9nbui.jpg)

1. [] In the Policy: Add or remove labels blade, check the Box next to **Legal Only** and click **OK**.

	!IMAGE[](Media/c2429kv9.jpg)

1. [] In the Policy blade, under **Configure settings to display and apply on Information Protection end users** section, under **Select the default label**, select **None** as the default label for this scoped policy.

	!IMAGE[4mxceage.jpg](Media/4mxceage.jpg)

1. [] Click **Save**, then **OK** to complete creation of the No Default Label Scoped Policy.

1. [] Click on the **X** in the upper right-hand corner to close the policy.

---

## Defining Recommended and Automatic Conditions

One of the most powerful features of Azure Information Protection is the ability to guide your users in making sound decisions around safeguarding sensitive data.  This can be achieved in many ways through user education or reactive events such as blocking emails containing sensitive data. 

However, helping your users to properly classify and protect sensitive data at the time of creation is a more organic user experience that will achieve better results long term.  In this task, we will define some basic recommended and automatic conditions that will trigger based on certain types of sensitive data.

1. [] Under **Classifications** on the left, click **Labels**.
1. [] Expand **Confidential**, and click on **Contoso Internal**.

	!IMAGE[](Media/jyw5vrit.jpg)
1. [] In the Label: Contoso Internal blade, scroll down to the **Configure conditions for automatically applying this label** section, and click on **+ Add a new condition**.

	!IMAGE[cws1ptfd.jpg](Media/cws1ptfd.jpg)
1. [] In the Condition blade, in the **Select information types** search Box, type ```EU``` and check the Boxes next to the **items shown below**.

	!IMAGE[xaj5hupc.jpg](Media/xaj5hupc.jpg)
1. [] Next, before saving, replace EU in the search bar with ```credit``` and check the Box next to **Credit Card Number**.

	!IMAGE[](Media/9rozp61b.jpg)
1. [] Click **Save** in the Condition blade and **OK** to the Save settings prompt.

	>  By default the condition is set to Recommended and a policy tip is created with standardized text.
	
	!IMAGE[qdqjnhki.jpg](Media/qdqjnhki.jpg)

1. [] Click **Save** in the Label: Contoso Internal blade and **OK** to the Save settings prompt.
1. [] Press the **X** in the upper right-hand corner to close the Label: Contoso Internal blade.
1. [] Next, expand **Highly Confidential** and click on the **All Employees** sub-label.

	!IMAGE[](Media/2eh6ifj5.jpg)
1. [] In the Label: All Employees blade, scroll down to the **Configure conditions for automatically applying this label** section, and click on **+ Add a new condition**.

	!IMAGE[](Media/8cdmltcj.jpg)
1. [] In the Condition blade, select the **Custom** tab and enter ```Password``` for the **Name** and in the textBox below **Match exact phrase or pattern**, type ```pass@word1```.

	!IMAGE[ra7dnyg6.jpg](Media/ra7dnyg6.jpg)
1. [] Click **Save** in the Condition blade and **OK** to the Save settings prompt.
1. [] In the Labels: All Employees blade, in the **Configure conditions for automatically applying this label** section, click **Automatic**.

   !IMAGE[245lpjvk.jpg](Media/245lpjvk.jpg)

	> The policy tip is automatically updated when you switch the condition to Automatic.

1. [] Click **Save** in the Label: All Employees blade and **OK** to the Save settings prompt.

1. [] Press the **X** in the upper right-hand corner to close the Label: All Employees blade.

---
## Review
 
In this section, we have created and modified labels, defined settings in global and scoped policies, and created recommended and automatic conditions to help protect our users.

In the next section, we will test these settings and see how they work with actual users.

---
===
# Testing AIP Policies

Now that you have users being affected by different policies, we can start testing these policies.  This section will run through various scenarios to demonstrate the use of AIP global and scoped policies and show the functionality of recommended and automatic labeling. 

---

## Testing User Defined permissions

One of the most common use cases for AIP is the ability to send emails using User Defined Permissions (Do Not Forward). In this task, we will send an email using the Do Not Forward label to test that functionality.


1. [] On VictimPC, log in using the credentials below

    >**JeffL**
    >
    >**Password$fun**	
1. [] Launch Microsoft Outlook, and click **Accept and start Outlook**.
1. [] In the username Box, type **MeganB@@lab.cloudcredentia139.tenantname** and click **Connect**.
1. [] When prompted, type the password **@lab.CloudCredential(139).Password** and Sign in.
5. On the Use this account everywhere page, click **Yes** then click **Done**.
1. [] Once configuration completes, **uncheck the Box** to **Set up Outlook Mobile** and click **OK**.
1. [] **Close Outlook** and **reopen** to complete activation.
1. [] Click on the **New email** button.

	> Note that the **Sensitivity** is set to **General** by default.
	
	!IMAGE[5esnhwkw.jpg](Media/5esnhwkw.jpg)

1. [] Send an email to **Adele Vance** and **Alex Wilber** (```Adele Vance;Alex Wilber```). You may also **optionally add an external email address** (preferably from a major social provider like gmail, yahoo, or outlook.com) to test the external recipient experience. 
1. [] For the **Subject** and **Body** type ```Test Do Not Forward Email```.

1. [] In the Sensitivity Toolbar, click on **Confidential** and then the **Do Not Forward** sub-label and click **Send**.

  	!IMAGE[w8j1w1lm.jpg](Media/w8j1w1lm.jpg)

	> If you receive the error message below, click on the Confidential \ Contoso Internal sub-label to force the download of your AIP identity certificates, then follow the steps above to change the label to Confidential \ Do Not Forward.
	
	!IMAGE[6v6duzbd.jpg](Media/6v6duzbd.jpg)

1. [] Switch over to Client01, log in using the credentials below

	>**LisaV**
	>
	>**HighImpactUser1!**
1. [] Open **Outlook**. 
1. [] Run through setup, this time using the credentials **AdeleV@@lab.cloudcredentia139.tenantname** and **@lab.CloudCredential(139).Password**. 
1. [] Review the email in Adele Vance’s Outlook.  You will notice that the email is automatically shown in Outlook natively.

	> If you elected to send a Do Not Forward message to an external email, you will have an experience similar to the images below.  These captures are included to demonstrate the functionality for those that chose not to send an external message.
	
	!IMAGE[tzj04wi1.jpg](Media/tzj04wi9.jpg)
	
	> Here the user has received an email from Megan Bowen and they can click on the **Read the message** button.
	
	!IMAGE[wiefwcho.jpg](Media/wiefwcho.jpg)
	
	>Next, the user is given the option to either log in using the social identity provider (**Sign in with Google**, Yahoo, Microsoft Account), or to **sign in with a one-time passcode**.
	>
	>If they choose the social identity provider login, it should use the token previously cached by their browser and display the message directly.
	>
	>If they choose one-time passcode, they will receive an email like the one below with the one-time passcode.
	
	!IMAGE[m6voa9xi.jpg](Media/M6voa9xi.jpg)
	
	>They can then use this code to authenticate to the Office 365 Message Encryption portal.
	
	!IMAGE[8pllxint.jpg](Media/8pllxint.jpg)
	
	>After using either of these authentication methods, the user will see a portal experience like the one shown below.
	
	!IMAGE[3zi4dlk1.jpg](Media/3zi4dlk9.jpg)

---

## Testing Global Policy

In this task, we will create a document and send an email to demonstrate the functionality defined in the Global Policy.

1. [] Switch to VictimPC and log in with the credentials below

    >**Jeffl**
    >
    >**Password$fun**
1. [] In Microsoft Outlook, click on the **New email** button.

1. [] Send an email to Adele Vance, Alex Wilber, and yourself (```Adele Vance;Alex Wilber;Your Email```).  
1. [] For the **Subject** and **Body** type ```Test Contoso Internal Email```.

1. [] In the Sensitivity Toolbar, click on **Confidential** and then **Contoso Internal** and click **Send**.


1. [] Switch to Client01 and log in using the credentials below

	>**LisaV**
	>
	>**HighImpactUser1!**
1. [] Observe that you are able to open the email natively in the Outlook client. Also observe the **header text** that was defined in the label settings.

	!IMAGE[bxz190x2.jpg](Media/bxz190x2.jpg)
	
1. [] In your email, note that you will be unable to open this message.  This experience will vary depending on the client you use (the image below is from Outlook 2016 for Mac) but they should have similar messages after presenting credentials. Since this is not the best experience for the recipient, later in the lab we will configure Exchange Online Mail Flow Rules to prevent content classified with internal only labels from being sent to external users.
	
	!IMAGE[52hpmj51.jpg](Media/52hpmj51.jpg)

---

## Testing Scoped Policy

In this task, we will create a document and send an email from one of the users in the Legal group to demonstrate the functionality defined in the first section. We will also show the behavior of the No Default Label policy on documents.

1. [] On Client01, in Microsoft Outlook, click on the **New email** button.
	
1. [] Send an email to Alex Wilber and Megan Bowen (```Alex Wilber;Megan Bowen```).  For the **Subject** and **Body** type ```Test Highly Confidential Legal Email```.
1. [] In the Sensitivity Toolbar, click on **Highly Confidential** and the **Legal Only** sub-label, then click **Send**.

1. [] Switch to AdminPC and log in with the credentials below

    >**SamiraA**
    >
    >**NinjaCat123!@#**
1. [] Run through setup, this time using the credentials **AlexW@@lab.cloudcredentia139.tenantname** and **@lab.CloudCredential(139).Password**. 
1. [] Review the email in Alex Wilber’s Outlook. You should be able to open the message natively in the client as Alex.

1. [] Switch to VictimPC and log in with the credentials below

    >**Jeffl**
    >
    >**Password$fun**
1. [] Click on the email. You should be unable to open the message as Megan.

1. [] Switch to Client01 and log in using the credentials below

	>**LisaV**
	>
	>**HighImpactUser1!**
1. [] Open **Microsoft Word**.
1. [] Create a new **Blank document** and type ```This is a test document``` and **save the document**.

	> When you click **Save**, you will be prompted to choose a classification.  This is a result of having **None** set as the default label in the scoped policy while requiring all documents to be labeled.  This is a useful for driving **active classification decisions** by specific groups within your organization.  Notice that Outlook still has a default of **General** because of the Advanced setting we added to the scoped policy.  **This is recommended** because user send many more emails each day than they create documents. Actively forcing users to classify each email would be an unpleasant user experience whereas they are typically more understanding of having to classify each document if they are in a sensitive department or role.

1. [] Choose a classification to save the document.

---

## Testing Recommended and Automatic Classification

In this task, we will test the configured recommended and automatic conditions we defined in section 1. []  Recommended conditions can be used to help organically train your users to classify sensitive data appropriately and provides a method for testing the accuracy of your dectections prior to switching to automatic classification.  Automatic conditions should be used after thorough testing or with items you are certain need to be protected. Although the examples used here are fairly simple, in production these could be based on complex regex statements or only trigger when a specific quantity of sensitive data is present.

1. [] Switch to VictimPC and log in with the credentials below

    >**Jeffl**
    >
    >**Password$fun**
1. [] Launch **Microsoft Word**.
1. [] In Microsoft Word, create a new **Blank document** and type ```My AMEX card number is 344047014854131. [] The expiration date is 09/28, and the CVV is 4368``` and **save** the document.

	> This card number is a fake number that was generated using the Credit Card Generator for Testing at [https://developer.paypal.com/developer/creditCardGenerator/](https://developer.paypal.com/developer/creditCardGenerator/).  The Microsoft Classification Engine uses the Luhn Algorithm to prevent false positives so when testing, please make sure to use valid numbers.

1. [] Notice that you are prompted with a recommendation to change the classification to Confidential \ Contoso Internal. Click on **Change now** to set the classification and protect the document.

	!IMAGE[url9875r.jpg](Media/url9875r.jpg)
	> Notice that, like the email earlier in this section, the header value configured in the label is added to the document.
	
	!IMAGE[dcq31lz1.jpg](Media/dcq31lz1.jpg)
1. [] In Microsoft Word, create a new **Blank document** and type ```my password is pass@word1``` and **save** the document.

	>Notice that the document is automatically classified and protected wioth the Highly Confidential \ All Employees label.
	
	!IMAGE[6vezzlnj.jpg](Media/6vezzlnj.jpg)
1. [] Next, in Microsoft Outlook, click on the **New email** button.
	
	
1. [] Draft an email to Alex Wilber and Adele Vance (```Alex Wilber;Adele Vance```).  For the **Subject** and **Body** type ```Test Highly Confidential All Employees Automation```.

1. [] Attach the **second document you created** to the email.

	!IMAGE[823tzyfd.jpg](Media/823tzyfd.jpg)

	> Notice that the email was automatically classified as Highly Confidential \ All Employees.  This functionality is highly recommended because matching the email classification to attachments provides a much more cohesive user experience and helps to prevent inadvertent information disclosure in the body of sensitive emails.
	
	!IMAGE[yv0afeow.jpg](Media/yv0afeow.jpg)

1. [] In the email, click **Send**.
   
---

## Review

In this section, you have tested Office 365 Message Encryption using the Do Not Forward label, tested the functionality of global and scoped policies to label, mark, and protect documents, and tested recommended and automatic conditions.

In the next section, we will take a look at the new unified labeling interface in the Security and Compliance Center.

---
===
## Bulk Classification

In this task, we will perform bulk classification using the built-in functionality of the AIP client.  This can be useful for users that want to classify/protect many documents that exist in a central location or locations identified by scanner discovery.  Because this is done manually, it is an AIP P1 feature.

1. [] Switch to AdminPC and log in with the credentials below

    >**SamiraA**
    >
    >**NinjaCat123!@#**
1. [] Browse to the **C:\\**.
1. [] Right-click on the PII folder and select **Classify and Protect**.
   
   !IMAGE[CandP.png](\Media/CandP.png)
1. [] If prompted, click use another account and use the credentials below to authenticate:

	>**AlexW@@lab.cloudcredential(139).tenantname** 
	>
	>**@lab.CloudCredential(139).Password**

1. [] In the AIP client Classify and protect interface, select **Highly Confidential\\All Employees** and press **Apply**. 

	!IMAGE[CandP2.png](\Media/CandP2.png)

> Warning: If you are unable to see the **Apply** button due to screen resolution, click **Alt+A** and **Enter** to apply the label to the content.

> Note: You may review the results in a text file by clicking show results, or simply close the window. 

---
===
# Security and Compliance Center

In this section, we will migrate your AIP Labels and activate them in the Security and Compliance Center.  This will allow you to see the labels in Microsoft Information Protection based clients such as Office 365 for Mac and Mobile Devices.

Although we will not be demonstrating these capabilities in this lab, you can use the tenant information provided to test on your own devices.

---
## Activating Unified Labeling

In this task, we will activate the labels from the Azure Portal for use in the Security and Compliance Center.

1. [] On Client01, in the AIP blade, click on **Unified labeling (Preview)**.

	!IMAGE[Unified Labeling](Media/Unified.png)

1. [] Click **Activate** and **Yes**.

	>NOTE: If this is already activated, please skip to the next section.

	!IMAGE[o0ahpimw.jpg](Media/o0ahpimw.jpg)

	>You should see a message similar to the one below.
	
	!IMAGE[SCCMigration.png](Media/SCCMigration.png) 

1. [] In a new tab, browse to ```https://protection.office.com/``` and click on **Classifications** and **Labels** to review the migrated labels. 

---
## Deploying Policy in SCC

The previous step enabled the AIP labels for use in the Security and Compliance Center.  However, this did not also recreate the policies from the AIP portal. In this step we will publish a Global policy like the one we used in the AIP portal for use with unified clients.

1. [] In the Security and Compliance Center, under Classifications, click on **Label policies**.

1. [] In the Label policies pane, click **Publish labels**.

	^IMAGE[Open Screenshot](Media/SCC01.png)
1. [] On the Choose labels to publish page, click the **Choose labels to publish** link.

	^IMAGE[Open Screenshot](Media/SCC02.png)
1. [] In the Choose labels pane, click the **+ Add** button.

	^IMAGE[Open Screenshot](Media/SCC03.png)
5. Click the box next to **Display name** to **select all labels**, then click the **Add** button.

	^IMAGE[Open Screenshot](Media/SCC04.png)
6. Click the **Done** button.

	^IMAGE[Open Screenshot](Media/SCC05.png)
7. Back on the Choose labels to publish page, click the **Next** button.

	^IMAGE[Open Screenshot](Media/SCC06.png)
8. On the Publish to users and groups page, notice that **All users** are included by default. If you were creating a scoped policy, you would choose specific users or groups to publish to. Click **Next**.

	^IMAGE[Open Screenshot](Media/SCC07.png)
9. On the Policy settings page, select the **General** label from the drop-down next to **Apply this label by default to documents and email**.
10. Check the box next to **Users must provide justification to remove a label or lower classification label** and click the **Next** button.

	^IMAGE[Open Screenshot](Media/SCC08.png)

11. [] In the Name textbox, type ```Global Policy``` and for the Description type ```This is the default global policy for all users.``` and click the **Next** button.

	^IMAGE[Open Screenshot](Media/SCC09.png)
11. [] Finally, on the Review your settings page, click the **Publish** button.

	^IMAGE[Open Screenshot](Media/SCC10.png)

---

## Review

In this section, we enabled and published labels and policies in the Security and Compliance Center for use with clients based on the MIP SDK.  This will allow Mac, Mobile, and other Microsoft and 3rd party applications to see the sensitivity labels stored in the Security and Compliance Center.

---
===
# Introduction to Azure Information Protection, Unified Labeling, and Office 365 Message Encryption Lab Complete

In this lab, you have learned about the basic elements of Azure Information Protection, Unified Labeling, and Office 365 Message Encryption.

You should now have knowledge of:

- How to use AIP to classify, label, protect, and control access to sensitive data
- Configuring labels and policy within the Azure portal
- Unified Labeling and how to configure and deploy labels within the Security and Compliance Center for use with Windows, Mac, Mobile, and 3rd party Unified Labeling clients
- The difference between the AIP client (classic) and the AIP Unified Labeling client and when each should be deployed
- How to configure recommended and automatic conditions in the Azure portal and the SCC
- Be able test manual, recommended, and automatic labeling using the AIP client on Windows systems
- Send email messages with Office 365 Message Encryption for secure collaboration to any recipient

Please refer to the other labs in this series for detail around deploying the AIP scanner and Advanced functionality of AIP.

!IMAGE[](Media/ninjacat.png)

---