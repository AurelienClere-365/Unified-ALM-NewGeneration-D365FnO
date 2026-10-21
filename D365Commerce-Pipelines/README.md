# Dynamics 365 Commerce: CI/CD pipelines

This folder contains two single-file Azure DevOps YAML pipelines. Each one builds and deploys in the same run.

They are templates. Values such as `CONTOSO`, the LCS project ID, the service connection name, the environment GUIDs and the environment names are placeholders to replace with your own.

**The templates are set up for 1 UAT and 1 PROD environment.** This is a starting point, not a constraint. Add, remove or rename environments to match your landscape (see section 7).

| Folder | Pipeline file | Builds | Deploys to |
|---|---|---|---|
| `ECommerce/` | `ecommerce-cicd-pipeline.yml` | e-Commerce extension (`yarn msdyn365 pack`) | LCS e-Commerce environment (or Dataverse) |
| `CommerceCSU/` | `csu-cicd-pipeline.yml` | Commerce Scale Unit extension (`CloudScaleUnitExtensionPackage.zip`) | LCS Commerce Scale Unit (or Dataverse) |

Each folder has its own README with the component-specific settings. This file covers what both pipelines share.

## 1. Branching model (both pipelines)

```
feature/xxx  --PR-->  uat  --PR-->  main
 (nothing)          Build + UAT    Build + PROD
```

| Event | Build | UAT | PROD |
|---|---|---|---|
| Commit on a personal branch (feature/..., bugfix/...) | No | No | No |
| PR into `uat` (build validation) | Yes | No | No |
| Commit or merged PR on `uat` | Yes | Yes | No |
| PR into `main` (must come from `uat`, otherwise the build fails) | Yes | No | No |
| Commit or merged PR on `main` | Yes | No | Yes, after approval |

Run names:
- `CONTOSO eCommerce <branch> <date>.<n>`, for example `CONTOSO eCommerce uat 20261021.1`.
- `CONTOSO CSU <branch> <date>.<n>`, for example `CONTOSO CSU main 20261021.3`.

Replace `CONTOSO` with your own prefix in the `name:` line and in the `buildPrefix` variable.

## 2. Shared prerequisites (do once)

### 2.1 Azure DevOps extension
Organization settings, Extensions: **Dynamics 365 finance and operations Tools** must be installed. It provides the LCS Asset Upload and LCS Asset Deployment tasks.

### 2.2 LCS service connection
Both pipelines use a **Dynamics Lifecycle Services** service connection (variable `lcsServiceConnection`, default `LCS-ServiceConnection`). If you do not have one yet:
- Create an app registration in Microsoft Entra ID with the API permission **Dynamics Lifecycle services, user_impersonation**, grant consent, and set **Allow public client flows = Yes** (Authentication, Advanced settings).
- Project settings, Service connections, New service connection, **Dynamics Lifecycle Services**. Enter the LCS API endpoint of your region, the user name and password of the account, and the application (client) ID.
- The connection signs in with a user name and password, so use a service account that is not subject to MFA prompts (check with your security team).
- The account must have the **Environment manager** role in the LCS project.
- The service connection name must stay written in the YAML. It cannot come from a variable group.
- On the first run of each pipeline, Azure DevOps asks for permission to use it. Click **Permit**.

### 2.3 Agent
- LCS validation of a package takes 45 to 50 minutes for e-Commerce. CSU servicing can take longer.
- A free Microsoft-hosted agent stops after 60 minutes. Use a paid Microsoft-hosted parallel job (up to 6 hours) or a self-hosted agent.
- With a self-hosted agent, replace `vmImage: 'windows-latest'` with `name: <your pool>`. The CSU build needs Visual Studio on the agent.

### 2.4 Azure DevOps Environments
Pipelines, Environments, New environment, resource **None**. The templates use four environments (1 UAT + 1 PROD per component). Create one per target environment:

| Environment | Checks |
|---|---|
| `eCommerce-UAT` | Exclusive lock (optional) |
| `eCommerce-PROD` | Approvals (release manager) + Exclusive lock |
| `CSU-UAT` | Exclusive lock (optional) |
| `CSU-PROD` | Approvals (release manager) + Exclusive lock |

## 3. Create a pipeline (for each component)

1. Copy the YAML file into the component repository, under `Pipeline/YAML_Files/`, and commit it directly on `main` (before the branch policies are active).
2. Create the `uat` branch from `main` (Repos, Branches, New branch).
3. Pipelines, New pipeline, Azure Repos Git, select the repository.
4. Existing Azure Pipelines YAML file, branch `main`, path `/Pipeline/YAML_Files/<file>.yml`.
5. Click the arrow next to Run, then **Save** (do not run yet).
6. Rename the pipeline, for example `CONTOSO eCommerce CICD` or `CONTOSO CSU CICD`.
7. Fill in the CONFIGURATION section (see the component README).
8. Validate the two LCS tasks with the task assistant (see the component README).
9. Disable the old build and release pipelines of that component once the new one works.

## 4. Branch policies and branch security (for each repository)

Repos, Branches, hover the branch, menu **...**, **Branch policies**. Configure `uat` first, then `main`.

### 4.1 Require a minimum number of reviewers
| Option | `uat` | `main` |
|---|---|---|
| Enabled | Yes | Yes |
| Minimum number of reviewers | 1 | 1 (or 2) |
| Allow requestors to approve their own changes | No | No |
| Prohibit the most recent pusher from approving their own changes | Yes | Yes |
| Allow completion even if some reviewers vote to wait or reject | No | No |
| When new changes are pushed | Reset all approval votes | Reset all approval votes |

### 4.2 Check for linked work items
Optional. Set to Required if every change must be linked to a Boards work item.

### 4.3 Check for comment resolution
**Required** on both branches.

### 4.4 Limit merge types
| Merge type | `uat` | `main` |
|---|---|---|
| Basic merge (no fast-forward) | Yes | **Yes, the only one allowed** |
| Squash merge | Yes | No |
| Rebase and fast-forward | No | No |
| Rebase with merge commit | No | No |

Squashing `uat` into `main` makes the two branches diverge, and every following PR shows false conflicts.

### 4.5 Build validation
Click **+** in Build Validation:
| Field | Value |
|---|---|
| Build pipeline | Your e-Commerce or CSU pipeline |
| Path filter | Empty (or the component folder if both components share a repository) |
| Trigger | Automatic |
| Policy requirement | Required |
| Build expiration | Immediately when the branch is updated |
| Display name | `Build eCommerce` or `Build CSU` |

The PR build compiles and packages but never deploys. On a PR into `main`, it fails when the source branch is not `uat`.

### 4.6 Automatically included reviewers
- On `main`: add the release manager (or a group) as **Required**.
- On `uat`: prefer a group with a minimum of 1 reviewer, or **Optional**. Individual required reviewers each have to approve, which blocks every UAT deployment until all of them vote.

### 4.7 Branch security
Repos, Branches, menu **...**, **Branch security**. For the developers group (for example Contributors):
| Permission | Value |
|---|---|
| Contribute | Inherited (Allow). The policies already block direct pushes. |
| Force push (rewrite history, delete branches) | **Not set** |
| Bypass policies when pushing | **Not set** |
| Bypass policies when completing pull requests | **Not set** |
| Manage permissions | Not set |

Not set is enough: without an Allow, the permission is denied. Do not set Deny on a broad group such as Contributors, because a Deny wins over an Allow given to a member of that group.

To give one or two administrators an emergency bypass: Branch security, add the person, **Bypass policies when completing pull requests = Allow**. The option **Override branch policies and enable merge** then appears under the Complete button of a PR.

### 4.8 Points of attention
- On a PR from `uat` into `main`, **uncheck "Delete source branch after merging"**, otherwise `uat` is deleted.
- On PRs from `feature/...` into `uat`, check it to clean up personal branches.
- An urgent fix also goes through `uat`, because a PR into `main` must come from `uat`. So `uat` must only contain what can go to production.
- Pipeline changes follow the same path: `FIX-xxx` into `uat`, then `uat` into `main`.

## 5. Test the flow (for each component)

| Test | Expected result |
|---|---|
| Commit on `feature/test` | Nothing is triggered |
| PR `feature/test` into `uat` | Build only |
| Merge into `uat` | Build, LCS upload, UAT deployment |
| PR `feature/test` into `main` | Build fails: "must come from uat" |
| PR `uat` into `main` | Build only |
| Merge into `main` | Build, approval, PROD deployment |

## 6. LCS asset type codes

In YAML, the `assetType` input of the LCS Asset Upload task and the `deploymentType` input of the LCS Asset Deployment task expect a code, not the label shown in the task assistant. Passing the label fails with `Cannot convert value "..." to type "System.Int32"` (upload) or `Selected Deployment Type '...' is not supported` (deploy). Both pipelines keep these codes in the `lcsAssetType` and `lcsDeploymentType` variables.

| Package | `assetType` (upload task) | `deploymentType` (deploy task) |
|---|---|---|
| e-Commerce package | `26` | `ecom` |
| Commerce Cloud Scale Unit extension | `29` | `csu` |

Other LCS asset type codes, for reference: 10 = Software deployable package, 28 = Retail self-service package.

## 7. Adapt the environments

The templates ship with **1 UAT and 1 PROD** environment per component. Change this to match your landscape. Each target environment is made of four pieces:

| Piece | Where | Example for UAT |
|---|---|---|
| Branch that deploys to it | `trigger:` + a `<key>Branch` variable | `uat`, `uatBranch` |
| Variables | CONFIGURATION section | `uatLcsEnvironmentId`, `uatEcommerceName` or `uatCsuName`, `uatAdoEnvironment` |
| Deploy stage | `stages:` | `Deploy_UAT` block |
| Azure DevOps Environment | Pipelines, Environments | `eCommerce-UAT` or `CSU-UAT` |

### 7.1 Add an environment with its own branch
Example: a TEST sandbox deployed from a `test` branch, before UAT (`feature` into `test` into `uat` into `main`).
1. Create the `test` branch and add it to `trigger: branches: include:`.
2. In CONFIGURATION, copy the UAT block and rename the prefix: `testBranch`, `testLcsEnvironmentId`, `testEcommerceName` (or `testCsuName`), `testAdoEnvironment`.
3. Copy the whole `Deploy_UAT` stage, rename it `Deploy_TEST`, set `envLabel` to `TEST`, and replace every `uat...` variable with `test...` (condition, environment, LCS inputs).
4. Create the Azure DevOps Environment (`eCommerce-TEST` or `CSU-TEST`).
5. Apply the branch policies of section 4 to `test`.
6. To force the promotion order, update the PR guard: a PR into `uat` must come from `test`. Copy the guard step and change the target and source branches.

### 7.2 Deploy several environments from the same branch
Example: UAT and a training sandbox both deployed from `uat`.
- Copy the `Deploy_UAT` stage as `Deploy_TRAINING`, keep the condition on `uatBranch`, and use new `training...` variables.
- Both stages depend on `Build`, so they run in parallel. To deploy one after the other, set `dependsOn: Deploy_UAT` on the second stage and add `succeeded()` in its condition.

### 7.3 Remove an environment
Delete its stage, its variables, its branch in `trigger:` and its Azure DevOps Environment. If you remove `uat`, also remove or adapt the PR guard in the Build stage.

### 7.4 Rename a branch
Branch names in `trigger:` cannot be variables. Change the name both in `trigger:` and in the matching `<key>Branch` variable.

### 7.5 Keep in mind
- A PROD deployment through LCS needs the approval on the PROD Azure DevOps Environment. Keep that check on every production target.
- LCS project IDs differ if your environments are in different LCS projects. In that case, turn `lcsProjectId` into per-environment variables (`uatLcsProjectId`, `prodLcsProjectId`) and use them in the stage of each environment.
