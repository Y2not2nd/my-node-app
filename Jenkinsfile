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
                        bat """
                            az logout || echo skipped
                            az login --service-principal ^
                              --username ${json.clientId} ^
                              --password ${json.clientSecret} ^
                              --tenant ${json.tenantId}

                            powershell Compress-Archive -Path * -DestinationPath app.zip
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
