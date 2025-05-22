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
                // Add error handling for npm install
                bat 'npm install || exit /b 1'
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([string(credentialsId: 'azure-sp', variable: 'AZURE_CREDENTIALS_JSON')]) {
                    script {
                        def json = readJSON text: AZURE_CREDENTIALS_JSON
                        env.clientId = json.clientId
                        env.clientSecret = json.clientSecret
                        env.tenantId = json.tenantId
                        env.subId = json.subscriptionId

                        // Added error handling and cleanup
                        bat """
                        echo ==== AZURE LOGIN ==== &&
                        az logout || echo Not logged in &&
                        az login --service-principal --username %clientId% --password %clientSecret% --tenant %tenantId% &&
                        az account set --subscription %subId% &&

                        echo ==== ZIPPING FULL PROJECT ==== &&
                        if exist app.zip del /f app.zip &&
                        powershell -Command "Compress-Archive -Path * -DestinationPath app.zip -Force -CompressionLevel Optimal -Exclude node_modules,.git,app.zip" &&

                        echo ==== CONFIGURE AZURE APP SETTINGS ==== &&
                        az webapp config appsettings set ^
                          --resource-group %AZURE_RG% ^
                          --name %AZURE_APP% ^
                          --settings WEBSITE_NODE_DEFAULT_VERSION=20 SCM_DO_BUILD_DURING_DEPLOYMENT=true &&

                        echo ==== DEPLOY TO AZURE APP SERVICE ==== &&
                        az webapp deploy ^
                          --resource-group %AZURE_RG% ^
                          --name %AZURE_APP% ^
                          --src-path app.zip ^
                          --type zip &&

                        del /f app.zip &&

                        IF %ERRORLEVEL% NEQ 0 (
                            echo Deployment failed
                            exit /b 1
                        )
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            // Cleanup
            bat 'az logout || echo Not logged in'
        }
    }
}