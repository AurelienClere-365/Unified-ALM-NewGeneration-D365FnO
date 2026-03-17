# Unified-ALM-NewGeneration-D365FnO

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

| File Name                          | Description |
|-----------------------------------|-------------|
| `xpp-ci.yml`                      | CI pipeline for compiling X++ code, running validations, and generating deployable packages for unified environments. |
| `xpp-build-validation.yml`        | Lightweight pipeline to validate X++ builds without deploying, useful for pull requests or early-stage testing. |
| `azure-build-pipelines-main.yml`  | LCS Mode : Main branch build pipeline for production-ready deployments, including full package generation and artifact publishing. |
| `azure-build-pipelines-test.yml`  | LCS Mode : Similar to the main pipeline but tailored for test/UAT environments, often with different variables or deployment targets. |
| `pac-modelbuilder-create-ude.yml` | Creates a Unified Developer Environment (UDE) using Power Platform CLI and model builder tools. |
| `pac-modelbuilder-manageenvironment-ude.yml` | Manages lifecycle tasks for UDEs, such as updates or configuration changes. |
| `pac.modelbuilder-delete.yml`     | Deletes a UDE environment cleanly from Power Platform. |
| `lcs-db-refresh-export.yml`       | Automates database refresh/export operations via Lifecycle Services (LCS) for sandbox or UAT environments. |
| `bpa-apps-dataverse.yml`          | Deploys Business Process Automation (BPA) apps to Dataverse environments using Power Platform Build Tools. |
| `process-update-dataverse.yml`    | Updates existing Dataverse solutions or components as part of a release pipeline. |
| `copy-environment.yml`            | Copies configuration or data from one environment to another, useful for cloning or templating. |
| `d365bap-tools.yml`               | Utility pipeline for deploying or managing D365 Business Application Platform (BAP) tools. |

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

## 📖 Learn More

For a full walkthrough and background on this project, check out the blog post:  
👉 [D365 FinOps DevOps and GitHub ALM New Generation](https://www.powerazure365.com/blog-1/d365-finops-devops-and-github-alm-new-generation)

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
