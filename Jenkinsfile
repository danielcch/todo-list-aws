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
        stage('Integration Tests') {
            steps {
                script {
                    withCredentials([
                        [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'jenkins_aws'],
                        string(credentialsId: 'aws_session_token', variable: 'AWS_SESSION_TOKEN')
                    ]) {
                        echo "Obteniendo URL del API Gateway desplegado..."
                        def baseUrl = sh(
                            script: """
                                aws cloudformation describe-stacks \
                                    --stack-name staging-todo-list-aws \
                                    --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                                    --region us-east-1 \
                                    --output text
                            """,
                            returnStdout: true
                        ).trim()

                        echo "BASE_URL obtenida: ${baseUrl}"

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
                        } else {
                            echo "integration-results.xml no encontrado. Saltando publicación."
                        }
                    }
                }
                failure {
                    echo "Tests de integración fallidos. Abortando pipeline."
                    error("Fase de integración fallida")
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
