# EntraOps (Privileged EAM) - Management and Monitoring of Enterprise Access Model
- [EntraOps (Privileged EAM) - Management and Monitoring of Enterprise Access Model](#entraops-privileged-eam---management-and-monitoring-of-enterprise-access-model)
  - [Introduction](#introduction)
  - [Key features](#key-features)
  - [Videos and demos of EntraOps Privileged EAM](#videos-and-demos-of-entraops-privileged-eam)
  - [Quickstarts](#quickstarts)
  - [Executing EntraOps interactively](#executing-entraops-interactively)
    - [Examples of using cmdlets and filter classification data](#examples-of-using-cmdlets-and-filter-classification-data)
  - [Using EntraOps with GitHub](#using-entraops-with-github)
  - [EntraOps Integration in Microsoft Sentinel](#entraops-integration-in-microsoft-sentinel)
    - [Parser for Custom Tables and WatchLists](#parser-for-custom-tables-and-watchlists)
    - [Examples to use EntraOps data in Unified SecOps Platform (Sentinel and XDR)](#examples-to-use-entraops-data-in-unified-secops-platform-sentinel-and-xdr)
    - [Workbook for visualization of EntraOps classification data](#workbook-for-visualization-of-entraops-classification-data)
  - [Classify privileged objects by Custom Security Attributes](#classify-privileged-objects-by-custom-security-attributes)
  - [Update EntraOps PowerShell Module and CI/CD (GitHub Actions)](#update-entraops-powershell-module-and-cicd-github-actions)
  - [Disclaimer and License](#disclaimer-and-license)

## Introduction

EntraOps is a personal research project to show capabilities for automated management of Microsoft Entra ID tenant at scale by using DevOps-approach. At this time, a PowerShell module and GitHub repository template is available to analyze privileges and use a (customizable) classification model to identify the sensitive of access (based on [Microsoft's Enterprise Access Model](https://aka.ms/SPA)). The solution can be used on any platform which supports PowerShell Core. Therefore, you have the option to run EntraOps in DevOps, serverless or local environments.

## Key features
- 🚀 Automation to configure and execute EntraOps in GitHub. Providing templates to execute EntraOps in Automation Accounts and Azure Functions are already planned.

- ☑️ Track changes and history of privileged principals and their assignments "as code"

- 🆕 Automation to update classification templates

- 👑 Automation to identify Control Plane scope by critical assets in Microsoft Security Exposure Management, high-privileges roles/scope in Microsoft Azure RBAC and privileged objects in Microsoft Entra (by EntraOps)

- 🔬 Ingest classification data with all details as enrichment to custom table in Microsoft Sentinel/Log Analytics Workspace or WatchLists

- 📊 Build reports or queries on your classified privileges to identify "tier breach" on Microsoft's Enterprise Access Model or privilege escalation paths. Workbook template to visualize classification data of role assignments (identified by EntraOps) and objects (by using custom security attributes)

- 🛡️ Automated coverage of privileged assets in Conditional Access and Restricted Management Administrative Units (in development) to protect high-privileged assets from lower privileges and apply strong Zero Trust policies.

Currently the following RBAC systems are supported:
- 🔑 Microsoft Entra roles
- 🔄 Microsoft Entra Identity Governance
- 🤖 Microsoft Graph App Roles

The following RBAC systems are in development and will be released soon:
- ☁️ Microsoft Azure (RBAC)
- 💵 Microsoft Billing Profiles (Enterprise Agreement)
- 🖥️ Microsoft Intune

EntraOps PowerShell module can be executed locally, as part of a CI/CD pipeline and any automation/worker environment which supports  PowerShell Core. The automation to create a pipeline supports GitHub only yet.

## Videos and demos of EntraOps Privileged EAM
- [TEC Talk: Protecting Privileged User and Workload Identities in Entra ID](https://www.quest.com/webcast-ondemand/tec-talk-protecting-privileged-user-and-workload-identities-in-entra-id/)
- [SpecterOps Webinar: Defining the Undefined: What is Tier Zero Part III](https://youtu.be/ykrse1rsvy4?si=f7fLcf1rAN0MGlti&t=1223)

## Quickstarts

## Executing EntraOps interactively

- Import PowerShell module
```powershell
Import-Module ./EntraOps
```
- Choose option to connect EntraOps with Microsoft Graph and Azure Resource Manager API:
  
*User Interactive with consented Microsoft Graph PowerShell*
```powershell
Connect-EntraOps -AuthenticationType "UserInteractive" -TenantName <TenantName>
```

*User Interactive in GitHub Codespaces with consented Microsoft Graph PowerShell*
```powershell
Connect-EntraOps -AuthenticationType "DeviceAuthentication" -TenantName <TenantName>
```

*Service Principal with ClientSecret*
```powershell
$ServicePrincipalCredentials = Get-Credential
Connect-AzAccount -Credential $ServicePrincipalCredentials -ServicePrincipal -Tenant $TenantName
Connect-EntraOps -TenantName $TenantName -AuthenticationType "AlreadyAuthenticated" -TenantName
```



### Examples of using cmdlets and filter classification data
- Export all classification of privileged objects with assignments to Entra ID directory roles and Microsoft Graph API permissions in EntraOps
```powershell
Save-EntraOpsPrivilegedEAMJson -RBACSystems @("EntraID", "ResourceApps")
```

- Save all Privileged Identities in Entra ID in a variable
```powershell
$PrivIdentities = Get-EntraOpsPrivilegedEamEntraId
```

- All Identities in Entra ID with Control Plane Permissions
```powershell
Get-EntraOpsPrivilegedEamEntraId | Where-Object { $_.RoleAssignments.Classification.AdminTierLevelName -contains "ControlPlane" } | ft
```

- Get a list of all privileged guests and applications
```powershell
Get-EntraOpsPrivilegedEamEntraId | Where-Object { $_.ObjectSubType -ne "Member" -and $_.ObjectType -ne "Group" } | ft
```

- List of all privileged hybrid identities
```powershell
Get-EntraOpsPrivilegedEamEntraId | Where-Object { $_.OnPremSynchronized -eq $true -and $_.RoleAssignments.Classification.AdminTierLevelName -contains "ControlPlane" }
```

- Display all privileged workload identities with Graph API permissions
```powershell
Get-EntraOpsPrivilegedEAMResourceApps
```


## Using EntraOps with GitHub
1. Create repository from this template
Choose private repository to keep data internal

2. Clone your new EntraOps repository to your client or use GitHub Codespace. Devcontainer is available to load the required dependencies.

3. Import EntraOps PowerShell Module in PowerShell Core
```powershell
Import-Module ./EntraOps
```

4. Create a new EntraOps.config File and update the settings based on your parameters and use case
_Tip: Use `Connect-AzAccount -UseDeviceAuthentication` before executing `New-EntraOpsConfigFile` if you are using GitHub Codespaces or Cloud Shell to perform Device Authentication._

```powershell
New-EntraOpsConfigFile -TenantName <TenantName>
```

5. Optional: Create data collection rule and endpoint if you want to ingest data to custom table in Log Analytics or Microsoft Sentinel workspace.
Follow the instructions from [Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/tutorial-logs-ingestion-portal#create-data-collection-endpoint) to configure a data collection endpoint, custom table and transformation rule.

    _Recommendation: There is a limitation of 10 KB for a single WatchList entry. This limit can be exceeded in the case of a high number of property items (e.g., classification or owner properties). Therefore, I can strongly recommend to choosing "Custom tables" in a large environment. If you are choosing WatchList as ingestion option, keep an eye on the deployment logs for any warnings of this limitation. Entries will not be added if the limit has been exceeded._

1. Review and customize the EntraOps.config file based on your requirements.
   * `TenantId` and `TenantName` should be already updated based on the provided parameters to create the config file. `ClientId` will be automatically updated by running the cmdlet `New-EntraOpsWorkloadIdentity`.
   * The default scheduled time for running the pull workflow will be a also enabled (`PullScheduledTrigger`) and defined (`PullScheduledCron`) in the config file. By default, the workflow to ingest data will be triggered right after the pull has been completed (by default value of `PushAfterPullWorkflowTrigger`).
   * Automated updates for classification templates from AzurePrivilegedIAM repository (`AutomatedClassificationUpdate`) or Control Plane scope (`ApplyAutomatedControlPlaneScopeUpdate`) can be also enabled by parameters. Customization of classification updates or data source to identify Control Plane assets is also available from here.
   * Review the settings in the section `AutomatedEntraOpsUpdate` to configure an automated update of the EntraOps PowerShell module on demand or scheduled basis.
   * Enable and update the following parameters if you want to ingest classification data to Custom Tables in Microsoft Sentinel/Log Analytics Workspace (`IngestToLogAnalytics`) or Microsoft Sentinel WatchLists (`IngestToWatchLists`). You need to add the required parameters of the workspace and/or data collection endpoints.

7. Create an application registration with required permissions (Global Admin role required)
```powershell
New-EntraOpsWorkloadIdentity -AppDisplayName entraops -CreateFederatedCredential -GitHubOrg "<YourGitHubUser/Org>" -GitHubRepo "EntraOps-<TenantName>" -FederatedEntityType "Branch" -FederatedEntityName "main"
```

8. Update GitHub workflow definition based on the definitions in EntraOps.config
```powershell 
Update-EntraOpsRequiredWorkflowParameters
```

## EntraOps Integration in Microsoft Sentinel
### Parser for Custom Tables and WatchLists
I have built a parser which ensures a standardized schema for EntraOps data across the various ingestion options.
This allows you to use the same queries and workbooks, regardless of whether you have used WatchLists or Custom Table.

Deploy the according parser for your ingestion option.
_Recommendation: Choose the parser for "Custom table" if you have enabled ingestion to both targets._

* **Parser for Custom Table (Microsoft Log Analytics/Sentinel Workspace)**

    [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCloud-Architekt%2FEntraOps%2Fmain%2FParsers%2FPrivilegedEAM_CustomTable.json)
    
* **Parser for Custom Table (Microsoft Sentinel Watchlists)**
  
    [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCloud-Architekt%2FEntraOps%2Fmain%2FParsers%2FPrivilegedEAM_WatchLists.json)


### Examples to use EntraOps data in Unified SecOps Platform (Sentinel and XDR)

**Devices in Exposure Management with authentication from Control Plane users**
```kusto
let ClassifiedTier0User = PrivilegedEAM
                | where Classification contains "ControlPlane"
                | where ObjectType == "user"
                | summarize arg_max(TimeGenerated, *) by ObjectId
                | project tostring(ObjectId), tostring(ObjectAdminTierLevel);
let Tier0Nodes = ExposureGraphNodes
                | where NodeLabel == "user"
                | mv-expand parse_json(EntityIds)
                | where parse_json(EntityIds).type == "AadObjectId"
                | extend NodeId = tostring(NodeId)
                | extend AadObjectId = tostring(parse_json(EntityIds).id)
                | extend TenantId = extract("tenantid=([\\w-]+)", 1, AadObjectId)
                | extend ObjectId = tostring(extract("objectid=([\\w-]+)", 1, AadObjectId))
                | where ObjectId in (ClassifiedTier0User);
let SensitiveRelation = dynamic(["can authenticate as","has credentials of","can authenticate as", "frequently logged in by"]);                
ExposureGraphEdges
| where TargetNodeId in (Tier0Nodes) and EdgeLabel in (SensitiveRelation)
| where SourceNodeLabel == "device"
// Get details of devices
| join kind = inner ( ExposureGraphNodes ) on $left.SourceNodeId == $right.NodeId
// Get ObjectId of Target (Tier0) Nodes
| join kind = inner ( Tier0Nodes ) on $left.TargetNodeId == $right.NodeId
| mv-expand parse_json(NodeProperties)
| summarize make_list(EdgeLabel) by SourceNodeName, SourceNodeLabel, tostring(SourceNodeCategories), TargetNodeId, TargetNodeName, TargetNodeLabel, tostring(TargetNodeCategories),
    VulnerableToPrivilegeEscalation = tostring(parse_json(tostring(parse_json(tostring(NodeProperties.rawData)).highRiskVulnerabilityInsights)).vulnerableToPrivilegeEscalation),
    MdeExpsoureScore = tostring(parse_json(NodeProperties).rawData.exposureScore),
    MdeRiskScore = tostring(parse_json(NodeProperties).rawData.riskScore),
    MdeSensorHealth = tostring(parse_json(NodeProperties).rawData.sensorHealthState),
    MdeMachineGroup = tostring(parse_json(NodeProperties).rawData.machineGroup)
```

**Resources with access or authentication to classified privileges in EntraOps**
```kusto
let SensitiveRelation = dynamic(["can authenticate as","has credentials of","affecting", "can authenticate as", "frequently logged in by"]);
let ClassifiedTier0Assets = PrivilegedEAM
                | summarize arg_max(TimeGenerated, *) by tostring(ObjectId);
let Tier0Nodes = ExposureGraphNodes
                | mv-expand parse_json(EntityIds)
                | where parse_json(EntityIds).type == "AadObjectId"
                | extend NodeId = tostring(NodeId)
                | extend AadObjectId = tostring(parse_json(EntityIds).id)
                | extend TenantId = extract("tenantid=([\\w-]+)", 1, AadObjectId)
                | extend ObjectId = extract("objectid=([\\w-]+)", 1, AadObjectId)
                | where ObjectId in (ClassifiedTier0Assets);
let ExposedEdges = ExposureGraphEdges
            | where EdgeLabel in (SensitiveRelation)
            | extend TargetNodeId = tostring(TargetNodeId)
            | join kind=inner ( Tier0Nodes ) on $left.TargetNodeId == $right.NodeId;
ClassifiedTier0Assets
| join kind=inner ( ExposedEdges ) on ObjectId
| project Type2, SourceNodeName, SourceNodeLabel, SourceNodeCategories, EdgeLabel, TargetNodeId, TargetNodeLabel, TargetNodeCategories, Classification, RoleAssignments, Categories
```

### Workbook for visualization of EntraOps classification data
The following Workbook can be used to check users, workload identities, groups, and their classified role assignments by EntraOps. It allows you also to filter for hybrid/cloud users and/or specific tiered administration level from by the classification of object or role assignments.

_Pre-requisite: EntraOps data has been ingested to WatchList or Custom Table and the associated Parser has been deployed._

* **EntraOps Privileged EAM - Overview**
  
  [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCloud-Architekt%2FEntraOps%2Fmain%2FWorkbooks%2FEntraOps%20Privileged%20EAM%20-%20Overview.json)


## Classify privileged objects by Custom Security Attributes
You might want to classify privileged users on the target Enterprise Access Level and relation to user/device. By default, the following custom security attributes will be used to identify what is the purposed tiered level of the user or workload identity.

- privilegedUser
- privilegedWorkloadIdentitiy

These attributes should be already set by the provisioning process. Check out the following blog posts to learn more about the integration:

- [Automated Lifecycle Workflows for Privileged Identities with Azure AD Identity Governance](https://www.cloud-architekt.net/manage-privileged-identities-with-azuread-identity-governance/)
- [Microsoft Entra Workload ID - Lifecycle Management and Operational Monitoring](https://www.cloud-architekt.net/entra-workload-id-lifecycle-management-monitoring/)

The purpsoed tiered level of a user or workload identity will be visible as attribute `ObjectAdminTierLevel` and `ObjectAdminTierLevelName` in the EntraOps data of the user principal.

In addition, custom security attributes will be also used to build a correlation between the privileged user and the associated PAW device and regular work account.
- associatedSecureAdminWorkstation
- associatedWorkAccount

## Update EntraOps PowerShell Module and CI/CD (GitHub Actions)
EntraOps can be updated without losing classification definition and files by using the cmdlet `Update-EntraOps`.
The cmdlet can be executed interactively, and changes must be pushed to your repository. This command updates the PowerShell module, workflow files and parameters in workflows based on "EntraOps.config" file.

Currently, there is also a workflow named "Update-EntraOps" which can be executed on demand or run on scheduled basis (defined in EntraOps.config) and updates the PowerShell module only.
I am working on a way to integrate the update process for the workflow files to the pipeline. There are some restrictions to update workflows by another workflow which needs to be handled.

## Disclaimer and License
This tool is provided as-is, with no warranties.
Code or documentation contributions, issue reports and feature requests are always welcome! 
Please use GitHub issue to review existing or create new issues.
The EntraOps project is MIT licensed.
