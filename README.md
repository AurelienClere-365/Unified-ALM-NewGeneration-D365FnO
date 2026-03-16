# Unifield-ALM-NewGeneration-D365FnO

A collection of Azure DevOps pipeline templates for implementing Application Lifecycle Management (ALM) in Dynamics 365 Finance & Operations (D365FnO) using the Unified Developer Experience and Power Platform integration.

## 🚀 Overview

This project provides ready-to-use YAML pipelines to automate CI/CD workflows for D365FnO and Power Platform solutions. It supports unified package generation, build validation, environment management, and deployment to both LCS and Power Platform Admin Center (PPAC) environments.

## ✨ Features

- Unified package generation for D365FnO and Power Platform
- CI/CD automation with Azure DevOps
- Build validation for X++ code
- Environment provisioning and management
- Dataverse and BPA integration

## 📦 Pipeline Summary

| File Name                          | Description                                      |
|-----------------------------------|--------------------------------------------------|
| `xpp-ci.yml`                      | CI pipeline for building and packaging X++ code  |
| `xpp-build-validation.yml`        | Validates X++ builds before deployment           |
| `azure-build-pipelines-main.yml`  | Main branch build pipeline in LCS mode                       |
| `azure-build-pipelines-test.yml`  | Test/UAT environment build pipeline in LCS mode             |
| `pac-modelbuilder-*.yml`          | Manage Power Platform model builder environments |
| `lcs-db-refresh-export.yml`       | Automate LCS database refresh/export             |
| `bpa-apps-dataverse.yml`          | BPA deployment to Dataverse                      |
| `process-update-dataverse.yml`    | Update Dataverse environments                    |
| `copy-environment.yml`            | Clone environment configurations                 |
| `d365bap-tools.yml`               | Utilities for D365 business apps                 |

## 🛠️ Getting Started

### Prerequisites

- Azure DevOps organization with required permissions
- Installed extensions:
  - [Dynamics 365 Finance and Operations Tools](https://marketplace.visualstudio.com/items?itemName=Dyn365FinOps.dynamics365-finops-tools)
  - [Power Platform Build Tools](https://learn.microsoft.com/en-us/power-platform/alm/devops-build-tools)
- X++ NuGet packages for your D365FnO version
- Service principal for Power Platform authentication

### Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/AurelienClere-365/Unifield-ALM-NewGeneration-D365FnO.git


## 🤝 Contributing

Contributions are welcome! Feel free to:

- Open issues for bugs or feature requests
- Submit pull requests with improvements
- Share feedback on pipeline configurations

---

## ⚠️ Notes

- Ensure all pipeline tasks (e.g., Create Deployable Package, Add Licenses) use version 1 to support unified package generation.
- If your models require licenses, include them in the `__License` folder or configure the licensing task accordingly.
- Power Platform deployments require a service principal with appropriate permissions (e.g., System Administrator role in the target environment).

---

Built for modern D365FnO developers embracing the Unified Developer Experience and Power Platform integration. 🚀
