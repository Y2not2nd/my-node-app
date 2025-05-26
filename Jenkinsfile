pipeline {
    agent any

    tools {
        nodejs 'Node 20'
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
                // Clean install
                bat 'npm ci || npm install'
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([string(credentialsId: 'azure-sp', variable: 'AZURE_CREDENTIALS_JSON')]) {
                    script {
                        def json = readJSON text: AZURE_CREDENTIALS_JSON
                        echo "DEBUG: Client ID = ${json.clientId}"
                        echo "DEBUG: Tenant ID = ${json.tenantId}"
                        echo "DEBUG: Subscription ID = ${json.subscriptionId}"
                        echo "DEBUG: App Name = ${env.AZURE_APP}"
                        echo "DEBUG: Resource Group = ${env.AZURE_RG}"

                        env.clientId = json.clientId
                        env.clientSecret = json.clientSecret
                        env.tenantId = json.tenantId
                        env.subId = json.subscriptionId

                        // Split commands for better error handling
                        bat """cmd /c ^
echo ==== AZURE LOGIN ==== && ^
az logout || echo Not logged in && ^
az login --service-principal --username %clientId% --password %clientSecret% --tenant %tenantId% --allow-no-subscriptions && ^
echo ==== VERIFY LOGIN ==== && ^
az account show && ^
echo ==== SET SUBSCRIPTION ==== && ^
az account set --subscription %subId% && ^
echo ==== VERIFY SUBSCRIPTION ==== && ^
az account show --query name -o tsv && ^
echo ==== PREPARE FOR DEPLOYMENT ==== && ^
if exist app.zip del /f app.zip && ^
echo ==== ZIPPING FILES ==== && ^
powershell -Command "Compress-Archive -Path server.js, package.json, package-lock.json, .deployment -DestinationPath app.zip -Force -Verbose" && ^
echo ==== VERIFY ZIP FILE ==== && ^
dir app.zip && ^
echo ==== CONFIGURE APP SETTINGS ==== && ^
az webapp config set --resource-group %AZURE_RG% --name %AZURE_APP% --startup-file "node server.js" --always-on true && ^
az webapp config appsettings set --resource-group %AZURE_RG% --name %AZURE_APP% --settings WEBSITE_NODE_DEFAULT_VERSION=~20 && ^
echo ==== DEPLOYING TO AZURE ==== && ^
az webapp deploy --resource-group %AZURE_RG% --name %AZURE_APP% --src-path app.zip --type zip --timeout 1800 --verbose && ^
echo ==== DEPLOYMENT COMPLETE ==== && ^
echo Checking deployment status... && ^
az webapp show --resource-group %AZURE_RG% --name %AZURE_APP% --query state -o tsv && ^
echo ==== CHECKING APPLICATION LOGS ==== && ^
az webapp log download --resource-group %AZURE_RG% --name %AZURE_APP% --log-file logs.txt && type logs.txt
"""
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo "==== Cleanup Phase ===="
                bat 'if exist app.zip del /f app.zip'
                bat 'if exist logs.txt del /f logs.txt'
                bat 'az logout || echo Already logged out'
            }
        }
    }
}