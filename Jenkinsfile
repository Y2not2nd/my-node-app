pipeline {
    agent any

    tools {
        nodejs 'Node 20' // Match the name you configured in Jenkins Global Tools
    }

    environment {
        AZURE_RG = 'test'                     // Your Azure Resource Group
        AZURE_APP = 'my-jenkins-demo-app'     // Your Web App name
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
                        env.clientId = json.clientId
                        env.clientSecret = json.clientSecret
                        env.tenantId = json.tenantId
                        env.subscriptionId = json.subscriptionId

                        bat """
                            echo ==== AZURE LOGIN ====
                            az logout || echo Not logged in
                            az login --service-principal ^
                              --username %clientId% ^
                              --password %clientSecret% ^
                              --tenant %tenantId%
                            az account set --subscription %subscriptionId%

                            echo ==== ZIPPING FILES ====
                            powershell -Command "Compress-Archive -Path * -DestinationPath app.zip -Force"

                            echo ==== DEPLOYING TO AZURE ====
                            az webapp deploy ^
                              --resource-group %AZURE_RG% ^
                              --name %AZURE_APP% ^
                              --src-path app.zip ^
                              --type zip
                        """
                    }
                }
            }
        }
    }
}
