pipeline {
    agent any
    environment {
        // Azure service principal credentials and app details
        AZURE_SP_ID     = '<YOUR-AZURE-APP-ID>'           // e.g. client/application ID of SP
        AZURE_TENANT    = '<YOUR-AZURE-TENANT-ID>'        // tenant ID
        AZURE_SP_SECRET = credentials('azure-sp')        // secret from Jenkins creds (Secret Text)
        AZURE_APP_NAME  = '<YOUR-WEBAPP-NAME>'           // Azure Web App name
        AZURE_GROUP     = '<YOUR-RESOURCE-GROUP-NAME>'    // Azure resource group name
    }
    stages {
        stage('Install Dependencies') {
            steps {
                // Install Node.js dependencies
                bat 'npm install'
            }
        }
        stage('Deploy to Azure') {
            steps {
                // Run Azure CLI commands in a single bat script with error handling
                bat """
                @echo off
                REM Check if Azure CLI is logged in (suppress output)
                az account show >nul 2>&1
                IF %ERRORLEVEL% EQU 0 (
                    echo Logging out any existing Azure CLI session...
                    az logout >nul 2>&1
                )
                echo Logging in to Azure with service principal...
                az login --service-principal -u \"%AZURE_SP_ID%\" -p \"%AZURE_SP_SECRET%\" --tenant \"%AZURE_TENANT%\" --output none
                IF %ERRORLEVEL% NEQ 0 (
                    echo ERROR: Azure CLI login failed.>&2
                    exit /b %ERRORLEVEL%
                )
                echo Azure CLI login succeeded.
                echo Packaging application into ZIP...
                powershell -NoLogo -Command \"Compress-Archive -Path '*' -DestinationPath 'app.zip' -Force\"
                IF %ERRORLEVEL% NEQ 0 (
                    echo ERROR: Failed to create deployment ZIP package.>&2
                    exit /b %ERRORLEVEL%
                )
                echo Deploying to Azure Web App \"%AZURE_APP_NAME%\" in resource group \"%AZURE_GROUP%\"...
                az webapp deploy --resource-group \"%AZURE_GROUP%\" --name \"%AZURE_APP_NAME%\" --src-path \"app.zip\" --type zip
                IF %ERRORLEVEL% NEQ 0 (
                    echo ERROR: Azure WebApp deployment failed.>&2
                    exit /b %ERRORLEVEL%
                )
                echo Deployment completed successfully.
                """
            }
        }
    }
}
