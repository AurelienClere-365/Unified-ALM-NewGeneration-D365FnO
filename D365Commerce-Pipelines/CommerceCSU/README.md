# Dynamics 365 Commerce Scale Unit (CSU) extension: CI/CD pipeline

File: `csu-cicd-pipeline.yml`. Place it in the CSU repository under `Pipeline/YAML_Files/`.

Shared setup (branching model, LCS connection, agent, Environments, branch policies, tests) is in the README at the root of this folder.

## What the pipeline does

1. **Build** (every run): runs `Pre-Build.ps1`, restores NuGet packages with `nuget.config`, builds the solution with Visual Studio, then publishes `CloudScaleUnitExtensionPackage.zip` as the `drop` artifact (renamed `ScaleUnitPackage_<branch>_<buildId>.zip`).
2. **Deploy_UAT** (commit or merged PR on `uat`): in LCS mode, uploads the package to the LCS Asset Library as a Commerce Cloud Scale Unit Extension and deploys it to the UAT CSU. In Dataverse mode, uploads it to the Dataverse environment of UAT.
3. **Deploy_PROD** (commit or merged PR on `main`): same as UAT, after the approval set on the `CSU-PROD` Environment.

The **Deployment mode** parameter (shown when you run the pipeline manually) selects `LCS` (default) or `Dataverse`. Automatic runs (commits and merges) use the default value. Change the default of `deploymentMode` in the YAML to switch for good.

The CSU package contains the extension code for the Commerce runtime (CRT) and Headless Commerce APIs, channel database scripts, the payment connector and Store Commerce for web.

Deploying a CSU extension replaces the previous one. If you have several CSU extensions, they must be merged into one package.

## Repository prerequisites

The repository follows the Microsoft sample **Dynamics365Commerce.ScaleUnit** (independent packaging model, Commerce SDK 10.0.19 and later), on the release branch that matches your Commerce version:
- A solution (`.sln`) that contains the extension projects and the ScaleUnit packaging project.
- `nuget.config` that references the `dynamics365-commerce` public feed and nuget.org.
- `Pipeline/PowerShellScripts/Pre-Build.ps1` and `Pipeline/PowerShellScripts/CopyFileToDestination.ps1` from the sample repository.
- Dataverse mode only: the upload script set in `dataverseUploadScript` (see "Dataverse mode" below).

## Variables (CONFIGURATION section at the top of the YAML)

The values below describe the template: 1 UAT and 1 PROD. To add or remove environments, see "Adapt the environments" in the root README.

| Variable | Current value | Description |
|---|---|---|
| `buildPrefix` | `CONTOSO CSU` | **To adapt**: prefix of the LCS asset and update names (also change it in the `name:` line) |
| `artifactName` | `drop` | Build artifact name |
| `solution` | `**\*.sln` | Solution to restore and build. Set the exact path if the repo has several solutions. |
| `buildConfiguration` | `Release` | Build configuration |
| `nugetConfigPath` | `nuget.config` | NuGet configuration file |
| `csuPackageSearchPath` | `$(Build.SourcesDirectory)\src\ScaleUnitSample` | **To adapt**: folder of your ScaleUnit packaging project. The zip is searched in it. |
| `csuPackageFile` | `CloudScaleUnitExtensionPackage.zip` | File produced by the packaging project |
| `lcsServiceConnection` | `LCS-ServiceConnection` | **To adapt**: name of your LCS service connection |
| `lcsProjectId` | `0000000` | **To fill in**: LCS project ID (end of the project dashboard URL) |
| `lcsAssetType` | `29` | LCS asset type code for a Commerce Cloud Scale Unit Extension |
| `uatBranch` | `refs/heads/uat` | Branch that deploys to UAT |
| `uatLcsEnvironmentId` | **To fill in** | UAT environment GUID (LCS, Environment details) |
| `uatCsuName` | **To fill in** | UAT scale unit name (LCS, Manage, Commerce, Commerce Scale Units) |
| `uatAdoEnvironment` | `CSU-UAT` | Azure DevOps Environment for UAT |
| `prodBranch` | `refs/heads/main` | Branch that deploys to PROD |
| `prodLcsEnvironmentId` | **To fill in** | PROD environment GUID |
| `prodCsuName` | **To fill in** | PROD scale unit name |
| `prodAdoEnvironment` | `CSU-PROD` | Azure DevOps Environment for PROD (with approval) |
| `uatDataverseVariableGroup` | `Commerce-CSU-UAT` | Dataverse mode only |
| `prodDataverseVariableGroup` | `Commerce-CSU-PROD` | Dataverse mode only |
| `dataverseUploadScript` | `Pipeline/PowerShellScripts/DataverseClient/UploadCsuExtensionPackage.ps1` | Dataverse mode only: upload script |
| `dataversePackageName` | `Contoso.Commerce.ScaleUnit` | **To adapt** (Dataverse mode): package name shown in Dataverse |
| `dataversePackageVersion` | `1.0.$(Build.BuildId)` | Dataverse mode: package version shown in Dataverse |

## Library (variable groups)

- **LCS mode: nothing to do.** LCS values are in the YAML and the credentials are in the service connection.
- **Dataverse mode:** create `Commerce-CSU-UAT` and `Commerce-CSU-PROD` with `DataverseUrl`, `TenantId`, `ClientId`, `ClientSecret` (lock icon) and `PackagePublisher`. If the e-Commerce pipeline already has Dataverse groups for the same environments, clone them. In each group, Pipeline permissions, add the CSU pipeline.

## Dataverse mode

### How it works
The Dataverse step uses the same approach as the e-Commerce pipeline. It creates a record in the Dataverse table `msprov_commerceextensionassets` and uploads the zip in chunks. The package is renamed `<dataversePackageName>-<dataversePackageVersion>.zip` before the upload, because the upload script reads the name and version from the file name when the zip has no `package.json` (a CSU package has none).

### Prerequisites
- An app registration with a client secret, added as an **Application User** in each target Dataverse environment, with write access to the `msprov_commerceextensionassets` table.
- The variable groups described above.

### The upload script (important)
Microsoft publishes a Dataverse upload script only for e-Commerce: `Pipeline/PowerShellScripts/DataverseClient/UploadExtensionPackage.ps1` in the **Msdyn365.Commerce.Online** sample. That script registers the package with the asset type **E-Commerce (1)**. There is no equivalent in the ScaleUnit sample at the time of writing. To use the Dataverse mode for the CSU:
1. Copy the whole `DataverseClient` folder from the Msdyn365.Commerce.Online sample into the CSU repository, under `Pipeline/PowerShellScripts/`.
2. Duplicate `UploadExtensionPackage.ps1` as `UploadCsuExtensionPackage.ps1`.
3. In the copy, replace the asset type value `1` (E-Commerce) with the value of the Commerce Scale Unit option of the asset type column. To find it: Power Apps, your environment, Tables, the Commerce extension asset table (`msprov_commerceextensionassets`), asset type column, list of choices.
4. Commit the script, then run the pipeline manually with **Deployment mode = Dataverse** on `uat` to test it.

If Microsoft later publishes a CSU script, point `dataverseUploadScript` to it and keep the same parameters.

### What the Dataverse mode does not do
Like the e-Commerce Dataverse mode, the step registers and uploads the package in Dataverse. Applying it to the scale unit then follows the Power Platform process of your environment.

## Validate the LCS tasks with the task assistant

Input names and picklist values depend on the version of the extension. In the pipeline editor, open **Show assistant**:
- **Dynamics Lifecycle Services Asset Upload** (v2): `assetType` must be `29`.
- **Dynamics Lifecycle Services Asset Deployment** (v4): select type **Commerce Cloud Scale Unit Extension**, enter the scale unit name, File Asset ID `$(lcsUpload.FileAssetId)`, Wait for completion checked, click **Add**. Copy the generated value of the deployment type and the exact name of the scale unit input into the YAML (the YAML uses `deploymentType` and `csuName` as placeholders), keeping the variables.
- Keep `name: lcsUpload` on the upload task.

## Points of attention

- **Timeout:** the deploy jobs allow 360 minutes (LCS validation and CSU servicing). This is the maximum on a Microsoft-hosted agent. A free agent stops after 60 minutes.
- **Downtime:** deploying to a CSU restarts the scale unit. Plan PROD deployments in a maintenance window. The approval on `CSU-PROD` lets you choose when the PROD stage starts.
- **SDK version:** keep the Commerce SDK packages referenced in the solution aligned with the version of the target environments.
- **Same repository as e-Commerce:** if both components live in one repository, uncomment the `paths: include` filter in both YAML files so that each pipeline only runs for its own folder, and set the same path filter in the Build validation policies.

## Troubleshooting

| Message | Cause | Fix |
|---|---|---|
| `Cannot convert value "..." to type "System.Int32"` | Label instead of code in `assetType` | Use `29` |
| Copy step fails, file not found | `csuPackageSearchPath` does not point to the packaging project | Set the folder of the ScaleUnit project |
| NuGet restore fails on Commerce packages | `nuget.config` does not reference the Commerce feed | Add the `dynamics365-commerce` feed from the sample |
| `A PR into refs/heads/main must come from refs/heads/uat` | PR from a personal branch into `main` | Target `uat` first, then PR `uat` into `main` |
| `This pipeline needs permission to access a resource` | First use of the service connection, variable group or Environment | Click **Permit** |
| Dataverse step: script not found | `UploadCsuExtensionPackage.ps1` not in the repository | See "The upload script" above |
| Package registered as E-Commerce in Dataverse | The original e-Commerce script is used without the asset type change | Change the asset type value in `UploadCsuExtensionPackage.ps1` |
