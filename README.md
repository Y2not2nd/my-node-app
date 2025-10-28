# My Node.js Azure Deployment Project

This project demonstrates how to deploy a Node.js Express app to Azure App Service using Jenkins automation.

The repository includes:
- A Node.js + Express app (`server.js`)
- A working Jenkins pipeline (`Jenkinsfile`) for automated builds and deployments
- Configuration files like `.deployment` and `package.json`
- Full setup instructions documented separately

---

##  Automated Deployment

The project is designed to be deployed using:
- Jenkins (with the provided `Jenkinsfile`)
- Azure App Service (Linux, Node.js stack)
- Azure Service Principal (for secure automation)

> **Note:**  
By default, the deployment uses:
- Resource Group: `test`
- App Service Name: `my-jenkins-demo-app`

If you use different names, update the Jenkinsfile and scripts accordingly.

---

##  Full Deployment Guide

For detailed step-by-step instructions, see:

 [`/docs/deployment_journey.md`](docs/deployment_journey.md)

This includes:
- How to set up Jenkins, plugins, and tools
- How to create the Resource Group and App Service in Azure
- How to configure Azure credentials in Jenkins
- How to troubleshoot builds and deployments

---

##  How to Check File History

To see older versions of files (like the Jenkinsfile):
1. Go to the file in GitHub.
2. Click **History**.
3. Browse past commits to review or restore changes.

---

##  Technologies Used

- Node.js + Express
- Azure App Service
- Jenkins Pipelines
- Azure CLI
- Git + GitHub
