pipeline {
    agent any

    tools {
        nodejs 'Node 20' // Must match Jenkins Global Tool Configuration
    }

    environment {
        AZURE_RG = 'test'
        AZURE_APP = 'my-jenkins-demo-app'
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
                        def creds = readJSON text: AZURE_CREDENTIALS_JSON

                        env.clientId     = creds.clientId
                        env.clientSecret = creds.clientSecret
                        env.tenantId     = creds.tenantId
                        env.subscription = creds.subscriptionId

                        bat '''
echo ==== AZURE LOGIN ====
az logout || echo Not logged in

az login --service-principal ^
  --username %clientId% ^
  --password %clientSecret% ^
  --tenant %tenantId%

az account set --subscription %subscription%

echo ==== ZIPPING FILES ====
powershell -Command "Compress-Archive -Path server.js,package.json,node_modules -DestinationPath app.zip -Force"

echo ==== DEPLOYING TO AZURE ====
az webapp deploy ^
  --resource-group %AZURE_RG% ^
  --name %AZURE_APP% ^
  --src-path app.zip ^
  --type zip || exit /b 1
'''
                    }
                }
            }
        }
    }
}
