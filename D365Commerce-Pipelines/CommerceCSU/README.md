# Dynamics 365 Commerce Scale Unit (CSU) extension: CI/CD pipeline

File: `csu-cicd-pipeline.yml`. Place it in the CSU repository under `Pipeline/YAML_Files/`.

Shared setup (branching model, LCS connection, agent, Environments, branch policies, tests) is in the README at the root of this folder.

## What the pipeline does

1. **Build** (every run): runs `Pre-Build.ps1`, restores NuGet packages with `nuget.config`, builds the solution with Visual Studio, then publishes `CloudScaleUnitExtensionPackage.zip` as the `drop` artifact (renamed `ScaleUnitPackage_<branch>_<buildId>.zip`).
2. **Deploy_UAT** (commit or merged PR on `uat`): uploads the package to the LCS Asset Library as a Commerce Cloud Scale Unit Extension and deploys it to the UAT CSU.
3. **Deploy_PROD** (commit or merged PR on `main`): same as UAT, after the approval set on the `CSU-PROD` Environment.

The CSU package contains the extension code for the Commerce runtime (CRT) and Headless Commerce APIs, channel database scripts, the payment connector and Store Commerce for web.

Deploying a CSU extension replaces the previous one. If you have several CSU extensions, they must be merged into one package.

## Repository prerequisites

The repository follows the Microsoft sample **Dynamics365Commerce.ScaleUnit** (independent packaging model, Commerce SDK 10.0.19 and later), on the release branch that matches your Commerce version:
- A solution (`.sln`) that contains the extension projects and the ScaleUnit packaging project.
- `nuget.config` that references the `dynamics365-commerce` public feed and nuget.org.
- `Pipeline/PowerShellScripts/Pre-Build.ps1` and `Pipeline/PowerShellScripts/CopyFileToDestination.ps1` from the sample repository.

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

## Library (variable groups)

Nothing to do. LCS values are in the YAML and the credentials are in the service connection. This pipeline has no Dataverse mode.

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
| `This pipeline needs permission to access a resource` | First use of the service connection or Environment | Click **Permit** |
