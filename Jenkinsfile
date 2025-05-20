pipeline {
    agent any

    tools {
        nodejs 'Node 20'  // Must match what you configured in Global Tool Configuration
    }

    environment {
        AZURE_RG = 'test'                     // Your actual resource group name
        AZURE_APP = 'my-jenkins-demo-app'     // Your Azure App Service name
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

                        // Debug output — REMOVE in production
                        echo "DEBUG: Client ID = ${json.clientId}"
                        echo "DEBUG: Tenant ID = ${json.tenantId}"
                        echo "DEBUG: Resource Group = ${env.AZURE_RG}"
                        echo "DEBUG: App Name = ${env.AZURE_APP}"

                        // Inject as environment variables for bat
                        env.clientId     = json.clientId
                        env.clientSecret = json.clientSecret
                        env.tenantId     = json.tenantId

                        bat '''
                            echo ============================
                            echo AZURE LOGIN
                            echo ============================

                            az account show 1>nul 2>&1
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

                            powershell -Command "Compress-Archive -Path * -DestinationPath app.zip -Force"

                            echo ============================
                            echo DEPLOYING TO AZURE WEB APP
                            echo ============================

                            az webapp deploy ^
                              --resource-group %AZURE_RG% ^
                              --name %AZURE_APP% ^
                              --src-path app.zip ^
                              --type zip

                            if %ERRORLEVEL% NEQ 0 (
                                echo Deployment failed!
                                exit /b 1
                            )
                        '''
                    }
                }
            }
        }
    }
}
