pipeline {
    agent { label 'agente1' }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/danielcch/todo-list-aws.git'
            }
        }
        stage('AWS SAM DEPLOY') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'jenkins_aws'],
                    string(credentialsId: 'aws_session_token', variable: 'AWS_SESSION_TOKEN')
                ]) {
                    sh '''
                        # Build
                        echo 'sam build'
                        sam build

                        # Validate
                        echo 'sam validate'
                        sam validate --region us-east-1

                        # Deploy
                        echo 'sam deploy en entorno: production'
                        sam deploy \
                            --config-env production \
                            --no-confirm-changeset \
                            --no-fail-on-empty-changeset
                    '''
                }
            }
        
        }
    }

    post {
        success {
            echo "Produccion desplegada correctamente desde master"
        }
        failure {
            echo "Error en el despliegue de produccion"
        }
    }
}
