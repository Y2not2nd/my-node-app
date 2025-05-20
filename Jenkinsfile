pipeline {
    agent any

    tools {
        nodejs 'Node 20'  // This must match the name you configured in Global Tools
    }

    environment {
        AZURE_RG = 'test'                     // Your actual resource group name
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

                        // Inject values into environment for use in bat
                        env.clientId     = json.clientId
                        env.clientSecret = json.clientSecret
                        env.tenantId     = json.tenantId

                        bat '''
                            echo ============================
                            echo Logging into Azure
                            echo ============================

                            rem logout safely even if not logged in
                            az account show >nul 2>&1
                            if %ERRORLEVEL%==0 (
                                az logout
                            ) else (
                                echo No account to log out from
                            )

                            az login --service-principal ^
                            --username %clientId% ^
                            --password %clientSecret% ^
                            --tenant %tenantId%
                            if %ERRORLEVEL% NEQ 0 (
                                echo Azure login failed!
                                exit /b 1
                            )

                            echo ============================
                            echo Creating zip package
                            echo ============================

                            powershell -Command "Compress-Archive -Path * -DestinationPath app.zip -Force"

                            echo ============================
                            echo Deploying to Azure Web App
                            echo ============================

                            az webapp deploy ^
                            --resource-group %AZURE_RG% ^
                            --name %AZURE_APP% ^
                            --src-path app.zip ^
                            --type zip
                            if %ERRORLEVEL% NEQ 0 (
                                echo Azure deploy failed!
                                exit /b 1
                            )
                        '''

                    }
                }
            }
        }
    }
}
