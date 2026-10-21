# Dynamics 365 Commerce e-Commerce extension: CI/CD pipeline

File: `ecommerce-cicd-pipeline.yml`. Place it in the e-Commerce repository under `Pipeline/YAML_Files/`.

It replaces the previous `build-pipeline.yml` and `release-pipeline.yml`. Shared setup (branching model, LCS connection, agent, Environments, branch policies, tests) is in the README at the root of this folder.

## What the pipeline does

1. **Build** (every run): Node.js, Yarn, `yarn install`, `yarn build`, `yarn msdyn365 pack`, then publishes the zip as the `drop` artifact.
2. **Deploy_UAT** (commit or merged PR on `uat`): uploads the package to the LCS Asset Library and deploys it to the UAT e-Commerce environment.
3. **Deploy_PROD** (commit or merged PR on `main`): same as UAT, after the approval set on the `eCommerce-PROD` Environment.

The **Deployment mode** parameter (shown when you run the pipeline manually) selects `LCS` (default) or `Dataverse`.

## Repository prerequisites

- `package.json` and `yarn.lock` at the repository root.
- `Pipeline/PowerShellScripts/CopyFileToDestination.ps1` (from the Msdyn365.Commerce.Online sample).
- Dataverse mode only: `Pipeline/PowerShellScripts/DataverseClient/UploadExtensionPackage.ps1`.

## Variables (CONFIGURATION section at the top of the YAML)

The values below describe the template: 1 UAT and 1 PROD. To add or remove environments, see "Adapt the environments" in the root README.

| Variable | Current value | Description |
|---|---|---|
| `buildPrefix` | `CONTOSO eCommerce` | **To adapt**: prefix of the LCS asset and update names (also change it in the `name:` line) |
| `nodeVersion` | `24.x` | Node.js version |
| `artifactName` | `drop` | Build artifact name |
| `lcsServiceConnection` | `LCS-ServiceConnection` | **To adapt**: name of your LCS service connection |
| `lcsProjectId` | `0000000` | **To fill in**: LCS project ID (end of the project dashboard URL) |
| `lcsAssetType` | `26` | LCS asset type code for an e-Commerce package |
| `uatBranch` | `refs/heads/uat` | Branch that deploys to UAT |
| `uatLcsEnvironmentId` | **To fill in** | UAT environment GUID (LCS, Environment details) |
| `uatEcommerceName` | **To fill in** | UAT e-Commerce environment name (LCS, Manage, Commerce, e-Commerce) |
| `uatAdoEnvironment` | `eCommerce-UAT` | Azure DevOps Environment for UAT |
| `uatDataverseVariableGroup` | `Commerce-ECommerce-UAT` | Dataverse mode only |
| `prodBranch` | `refs/heads/main` | Branch that deploys to PROD |
| `prodLcsEnvironmentId` | **To fill in** | PROD environment GUID |
| `prodEcommerceName` | **To fill in** | PROD e-Commerce environment name |
| `prodAdoEnvironment` | `eCommerce-PROD` | Azure DevOps Environment for PROD (with approval) |
| `prodDataverseVariableGroup` | `Commerce-ECommerce-PROD` | Dataverse mode only |

Where to find the values:
- Environment GUIDs: LCS, environment page, Environment details. If you had a classic release pipeline, the GUID is often stored in one of its variables.
- e-Commerce environment names: LCS, environment page, Manage, Commerce, e-Commerce.

## Library (variable groups)

- **LCS mode: nothing to do.** LCS values are in the YAML and the credentials are in the service connection.
- **Dataverse mode (later):** clone your existing release variable group (the Microsoft sample names it `Commerce-ECommerce-Extension-Release-Variables`) into `Commerce-ECommerce-UAT` and `Commerce-ECommerce-PROD`, with `DataverseUrl`, `TenantId`, `ClientId`, `ClientSecret` (lock icon), `PackagePublisher` and optionally `PackageName`. In each group, Pipeline permissions, add the e-Commerce pipeline.

## Validate the LCS tasks with the task assistant

Input names and picklist values depend on the version of the extension. In the pipeline editor, open **Show assistant**:
- **Dynamics Lifecycle Services Asset Upload** (v2): already validated. `assetType` must be `26`.
- **Dynamics Lifecycle Services Asset Deployment** (v4): select type **e-commerce Package**, File Asset ID `$(lcsUpload.FileAssetId)`, Wait for completion checked, click **Add**. Copy the generated value of the deployment type and the exact name of the e-Commerce environment input into the YAML, keeping the variables.
- Keep `name: lcsUpload` on the upload task. The deploy task reads the asset ID from it.
- Version 4 of the deploy task no longer needs the MSAL.PS install task.

## Troubleshooting

| Message | Cause | Fix |
|---|---|---|
| `Cannot convert value "e-Commerce package" to type "System.Int32"` | Label instead of code in `assetType` | Use `26` |
| `A PR into refs/heads/main must come from refs/heads/uat` | PR from a personal branch into `main` | Target `uat` first, then PR `uat` into `main` |
| `This pipeline needs permission to access a resource` | First use of the service connection or Environment | Click **Permit** |
| `X required reviewers must approve` | Automatically included reviewers set as Required on the branch | Get the approvals, or change the policy (root README, 4.6) |
| Job canceled after 60 minutes | Free hosted agent | Paid parallel job or self-hosted agent |
