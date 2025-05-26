### Jenkins Deployment Guide

#### Overview
This guide explains how to:
- Deploy Jenkins on a project.
- Configure Jenkins for automated deployment.
- Create a Jenkinsfile.

---

### What You Need
Before you begin, ensure you have the following prerequisites in place:
- An existing project (e.g., a simple Node.js application) that you want to deploy.
- Node.js installed on your local machine.
- Azure CLI installed on your local machine for interacting with Azure services.
- An active Azure Subscription.
- An Azure App Service named `my-jenkins-app`. If you don't have one, you'll need to create it as described in the Azure Setup section.
- A patient environment (this refers to the specific naming conventions for your resources, which might vary). Be sure to update the Jenkinsfile and any Azure resources to match your environment's naming.

---

### Project Layout
This section describes the typical structure for your project files, which will be used by Jenkins for deployment.

- **Node.js project**: This is your application's source code, typically set up manually.
- **Simple API**: A basic API that runs locally for testing purposes.
- **Jenkins**: The Jenkins automation server, which should be installed locally.
- **Git**: The version control system, installed locally on your machine.
- **Azure**: Your local Azure CLI setup and the necessary Azure cloud resources.

---

### Project Folder Structure
The recommended folder structure for your project is as follows:

- .deployment: This file defines the Azure startup command.
- Jenkinsfile: This file contains the Jenkins pipeline steps (placed directly at the project root, not inside a folder).
- package.json: This file lists your Node.js application's dependencies.
- package-lock.json: This file locks the exact versions of your Node.js dependencies.
- server.js: Your main Node.js server file.
- README.md: Project documentation.
- docs/deployment_journey.md: Detailed deployment instructions.
- node_modules/: Local dependencies (excluded from Git).

  The following files are typically created manually by you:

- .deployment
- Jenkinsfile
- package.json
- package-lock.json
- server.js
- README.md
- docs/deployment_journey.md

---

### Local Setup
Follow these steps to set up your project on your local development machine.

1. **Create the project folder**  
   Run these commands in your local terminal:  
   ```
   mkdir my-project
   cd my-project
   mkdir deployment
   ```
   This creates the main project directory and a subdirectory for deployment-related files.

2. **Initialize npm for your project**  
   Run this command in your local terminal:  
   ```
   npm init
   ```
   Press Enter repeatedly to accept the default values for the project's package.json file.

3. **Install the Express.js framework**  
   Run this command in your local terminal:  
   ```
   npm install express --save
   ```
   This installs Express.js, a popular Node.js web framework, and adds it to your project's dependencies.

4. **Create the `server.js` file**  
   Create a new file named `server.js` inside your `my-project` directory and add the following code. This file will serve as a basic web server:
   ```javascript
   const express = require('express');
   const app = express();
   const port = process.env.PORT || 3000;

   app.get('/', (req, res) => {
     res.send('Hello from your app, deployed via Jenkins!');
   });

   app.listen(port, () => {
     console.log('App listening on port ' + port);
   });
   ```

5. **Test the application (optional)**  
   Run this command in your local terminal to start your server:  
   ```
   node server.js
   ```
   You should see the message `App listening on port 3000` in your terminal. Open your web browser and navigate to `http://localhost:3000`. You should see the message `Hello from your app, deployed via Jenkins!` in your browser. This confirms your basic server is working.

6. **Create the `index.js` file**  
   Run this command in your local terminal:  
   ```
   touch index.js
   ```
   This creates an empty `index.js` file, which is often a placeholder for the main application logic.

---

### GitHub Setup
These steps guide you through setting up your GitHub repository for version control.

1. **Create a new GitHub repository**  
   Open your web browser and go to `https://github.com/new`. Follow the prompts to create a new, empty repository.

2. **Initialize a Git project in your local directory**  
   In your local terminal, navigate to your `my-project` directory and run:  
   ```
   git init
   ```
   This command initializes a new Git repository in your current directory.

3. **Add the remote origin for your GitHub repository**  
   In your local terminal, run:  
   ```
   git remote add origin https://github.com/YOUR_USERNAME/my-node-app.git
   ```
   Replace `YOUR_USERNAME` with your actual GitHub username and `my-node-app` with your repository name. This links your local Git repository to your GitHub repository.

4. **Add and commit your project files**  
   In your local terminal, run:  
   ```
   git add .
   git commit -m "Initial commit"
   ```
   The `git add .` command stages all changes in your current directory, and `git commit` saves these staged changes to your local repository with a descriptive message.

5. **Push your changes to the main branch on GitHub**  
   In your local terminal, run:  
   ```
   git push -u origin main
   ```
   This command uploads your local commits to the `main` branch of your GitHub repository. The `-u` flag sets the upstream branch, so future `git push` commands can be run without specifying `origin main`.

---

### Azure Setup
Follow these steps to configure your necessary Azure cloud resources using the Azure CLI.

1. **Azure CLI setup (if not already done)**  
   Run this command in your local terminal with Azure CLI installed:  
   ```
   az login
   ```
   This command will open a browser window for you to log in to your Azure account.

2. **Create a resource group**  
   Run this command in your local terminal:  
   ```
   az group create --name my-jenkins-rg --location westeurope
   ```
   This creates a new Azure resource group named `my-jenkins-rg` in the `westeurope` region.

3. **Create an Azure App Service plan and web app**  
   Run these commands in your local terminal:  
   ```
   az appservice plan create --name my-jenkins-plan --resource-group my-jenkins-rg --sku B1 --is-linux
   az webapp create --name my-jenkins-app --resource-group my-jenkins-rg --plan my-jenkins-plan
   ```

4. **Create an Azure Service Principal (if needed)**  
   Run this command in your local terminal:  
   ```
   az ad sp create-for-rbac --name http://my-jenkins-sp --role contributor --scopes /subscriptions/YOUR_SUBSCRIPTION_ID
   ```
   Replace `YOUR_SUBSCRIPTION_ID` with your actual Azure subscription ID. This command creates an Azure Active Directory application and service principal, which Jenkins will use to authenticate and deploy to your Azure resources.

---

### Running Builds and Troubleshooting

#### Run Jenkins Build
1. Go to Jenkins → your job → Build Now.
2. Check Build History → latest build → Console Output.
   - Green check = success.
   - Red X = failure.

#### Check Azure Logs
Run the following command to monitor logs:
```
az webapp log tail --resource-group YOUR_RG --name YOUR_APP
```
Monitor for:
- Startup failures.
- Runtime crashes.
- Configuration issues.

#### Check File History in GitHub
To review previous versions of the Jenkinsfile or other files:
1. Go to GitHub → repository → file.
2. Click **History**.
3. Browse and compare commits.
4. Restore or reuse past configurations if needed.

---

### Common Errors and Solutions

| **Error**                     | **Cause**                                | **Fix**                                                                 |
|-------------------------------|------------------------------------------|-------------------------------------------------------------------------|
| MODULE_NOT_FOUND: express     | Missing `node_modules` on App Service    | Let Azure install dependencies using `package.json` and `package-lock.json`. |
| Deployment timeout            | Zipping too many files, especially `node_modules` | Only zip source files; let Azure handle installs.                      |
| Jenkins fails to `az login`   | Incorrect Service Principal or Jenkins config | Double-check Azure credentials and Jenkins setup.                      |
| App Service shows Application Error | Wrong startup command or environment variables | Set correct startup file (`node server.js`) and ensure `PORT` is used dynamically. |
