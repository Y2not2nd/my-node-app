pipeline {
    agent any

    tools {
        nodejs 'Node 20' // Must match the name configured in Jenkins global tools
    }

    environment {
        AZURE_RG = 'test'                     // Your Azure resource group
        AZURE_APP = 'my-jenkins-demo-app'     // Your Azure Web App name
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

                        // Debug logging (safe unless you're livestreaming this)
                        echo "DEBUG: Client ID = ${json.clientId}"
                        echo "DEBUG: Tenant ID = ${json.tenantId}"
                        echo "DEBUG: Resource Group = ${env.AZURE_RG}"
                        echo "DEBUG: App Name = ${env.AZURE_APP}"

                        // Inject as env vars for batch access
                        env.clientId     = json.clientId
                        env.clientSecret = json.clientSecret
                        env.tenantId     = json.tenantId

                        // Safely wrapped in cmd /c to avoid Jenkins early failure
                        bat '''
cmd /c "
    echo ============================
    echo AZURE LOGIN
    echo ============================

    az account show >nul 2>&1
    if %ERRORLEVEL%==0 (
        echo Already logged in. Logging out...
        az logout
    ) else (
        echo No active Azure session.
    )

    echo Logging in with Service Principal...
    echo Client ID: %clientId%
    echo Tenant ID: %tenantId%

    az login --service-principal ^
      --username %clientId% ^
      --password %clientSecret% ^
      --tenant %tenantId%

    if %ERRORLEVEL% NEQ 0 (
        echo Azure login failed!
        exit /b 1
    )

    echo ============================
    echo CREATING ZIP PACKAGE
    echo ============================

    powershell -Command \\"Compress-Archive -Path * -DestinationPath app.zip -Force\\"

    echo ============================
    echo DEPLOYING TO AZURE WEB APP
    echo ============================

    az webapp deploy ^
      --resource-group %AZURE_RG% ^
      --name %AZURE_APP% ^
      --src-path app.zip ^
      --type zip

    if %ERRORLEVEL% NEQ 0 (
        echo Deploy failed!
        exit /b 1
    )
"
'''
                    }
                }
            }
        }
    }
}
