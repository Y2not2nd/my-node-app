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

                        bat 'echo ==== AZURE LOGIN ===='
                        bat 'az logout || echo Not logged in'
                        bat """
                            az login --service-principal ^
                              --username %clientId% ^
                              --password %clientSecret% ^
                              --tenant %tenantId%
                        """
                        bat 'az account set --subscription %subscriptionId%'

                        bat 'echo ==== ZIPPING FILES ===='
                        bat 'powershell -Command "Compress-Archive -Path * -DestinationPath app.zip -Force"'

                        bat 'echo ==== DEPLOYING TO AZURE ===='
                        bat """
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
