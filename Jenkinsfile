pipeline {
    agent { label 'agente1' }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {
        stage('Limpiando Workspace') {
            steps {
                echo 'Cleaning workspace...'
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/danielcch/todo-list-aws.git',
                    credentialsId: 'GitHub_token'
            }
        }

        stage('Despliegue Producción') {
            steps {
                echo 'Desplegando en producción con AWS SAM...'
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'jenkins_aws'],
                    string(credentialsId: 'aws_session_token', variable: 'AWS_SESSION_TOKEN')
                ]) {
                    sh '''
                        sam build
                        sam validate --region us-east-1
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
            echo 'Despliegue en producción completado correctamente.'
        }
        failure {
            echo 'Fallo en el despliegue de producción.'
        }
        always {
            cleanWs()
        }
    }
}
