pipeline {
    agent { label 'agente1' }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {
        stage('Checkout Master') {
            steps {
                checkout scm
                script {
                    def branchName = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    echo "Rama actual detectada: ${branchName}"
                    if (branchName != "master") {
                        error("No estamos en master, se para el pipeline.")
                    }
                }
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
