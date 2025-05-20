pipeline {
    agent any

    tools {
        nodejs 'Node 20' // Must match name in Jenkins Global Tool Configuration
    }

    environment {
        AZURE_RG  = 'test'                 // Your Azure resource group
        AZURE_APP = 'my-jenkins-demo-app' // Your Azure Web App name
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Code already checked out by Jenkins'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([string(credentialsId: 'azure-sp', variable: 'AZURE_CREDENTIALS_JSON')]) {
                    script {
                        def json = readJSON text: AZURE_CREDENTIALS_JSON

                        // Extract and export values
                        env.clientId     = json.clientId
                        env.clientSecret = json.clientSecret
                        env.tenantId     = json.tenantId
                        env.subId        = json.subscriptionId

                        // Start deployment
                        bat """
echo ============================
echo AZURE LOGIN
echo ============================

az logout || echo Not logged in

az login --service-principal ^
  --username %clientId% ^
  --password %clientSecret% ^
  --tenant %tenantId%

if %ERRORLEVEL% NEQ 0 (
    echo Azure login failed
    exit /b 1
)

az account set --subscription %subId%

echo ============================
echo CREATING ZIP PACKAGE
echo ============================

powershell -Command "Compress-Archive -Path * -DestinationPath app.zip -Force"

echo ============================
echo DEPLOYING TO AZURE
echo ============================

az webapp deploy ^
  --resource-group %AZURE_RG% ^
  --name %AZURE_APP% ^
  --src-path app.zip ^
  --type zip

if %ERRORLEVEL% NEQ 0 (
    echo Deployment failed
    exit /b 1
)
"""
                    }
                }
            }
        }
    }
}
