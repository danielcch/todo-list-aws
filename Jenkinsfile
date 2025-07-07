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

        stage('Get Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/danielcch/todo-list-aws.git',
                    credentialsId: 'GitHub_token'
            }
        }

        stage('Deploy') {
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
        stage('Estata test de integracion') {
            steps {
                echo 'Lanzando test de integracion...'
                script {
                    withCredentials([
                        [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'jenkins_aws'],
                        string(credentialsId: 'aws_session_token', variable: 'AWS_SESSION_TOKEN')
                    ]) {
                        echo "Obteniendo URL stack generado..."
                        def baseUrl = sh(
                            script: '''
                                aws cloudformation describe-stacks \
                                    --stack-name todo-list-aws-production \
                                    --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                                    --region us-east-1 \
                                    --output text
                            ''',
                            returnStdout: true
                        ).trim()

                        echo "BASE_URL: ${baseUrl}"

                        withEnv(["BASE_URL=${baseUrl}"]) {
                            echo "Ejecutando tests de integración contra ${baseUrl}"
                            sh '''
                                pytest --junitxml=integration-results.xml test/integration/todoApiTest.py || exit 1
                            '''
                        }
                    }
                }
            }
            post {
                always {
                    script {
                        if (fileExists('integration-results.xml')) {
                            junit 'integration-results.xml'
                            archiveArtifacts artifacts: 'integration-results.xml', fingerprint: true
                        } else {
                            echo "integration-results.xml no encontrado, no se publica."
                        }
                    }
                }
                failure {
                    echo "Tests de integracion fallidos. Abortando pipeline."
                    error("Fase de integracion fallida")
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline terminado.'
            cleanWs()
        }
        success {
            echo 'Despliegue en producción completado correctamente.'
        }
        failure {
            echo 'Fallo en el despliegue de producción.'
        }
    }
}
