pipeline {
    agent any

    environment {
        AZURE_RG = 'myResourceGroup'
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
                sh 'npm install'
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([string(credentialsId: 'azure-sp', variable: 'AZURE_CREDENTIALS_JSON')]) {
                    writeFile file: 'azurecreds.json', text: AZURE_CREDENTIALS_JSON
                    sh '''
                        az logout || true
                        az login --service-principal \
                          --username $(jq -r .clientId azurecreds.json) \
                          --password $(jq -r .clientSecret azurecreds.json) \
                          --tenant $(jq -r .tenantId azurecreds.json)

                        zip -r app.zip .
                        az webapp deploy \
                          --resource-group $AZURE_RG \
                          --name $AZURE_APP \
                          --src-path app.zip \
                          --type zip
                    '''
                }
            }
        }
    }
}
