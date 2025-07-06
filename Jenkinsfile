pipeline {
    agent { label 'agente1' }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'develop',
                    url: 'https://github.com/danielcch/todo-list-aws.git',
                    credentialsId: 'GitHub_token' //con to ken para luego hacer push
            }
        }

        stage('Unit Tests') {
            environment {
                DYNAMODB_TABLE = 'todoTableTest'
            }
            steps {
                sh '''
                    export PYTHONPATH=$PYTHONPATH:$(pwd)
                    pytest --cov=src --cov-report=term --cov-report=xml --cov-report=html  --junitxml=unit-results.xml ./test/unit/Test*.py
                '''
                junit '**/unit-results.xml'
            }
        }

        stage('Coverage') {
            environment {
                DYNAMODB_TABLE = 'todoTableTest'
            }  
            steps {
                sh '''
                    python3 -m coverage html
                '''
                publishHTML(target: [
                    reportDir: 'htmlcov',
                    reportFiles: 'index.html',
                    reportName: 'Informe de Cobertura',
                    keepAll: true
                ])
                
                cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,40', lineCoverageTargets: '100,0,40'

            }
        }
        stage('Static') {
            steps {
                sh '''
                    python3 -m flake8 --exit-zero --format=pylint src > flake8.out || echo "No se encontraron problemas" > flake8.out
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                qualityGates: [[threshold:10, type: 'TOTAL', unstable: true], 
                [threshold: 11, type: 'TOTAL', unstable: false]]
            }
        }

        stage('Security') {
            steps{
                sh '''
                    python3 -m bandit -r src -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}" || true
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], 
                qualityGates: [[threshold:1, type: 'TOTAL', unstable: true], 
                [threshold: 2, type: 'TOTAL', unstable: false]]
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
                        echo 'sam deploy en entorno: staging'
                        sam deploy \
                            --config-env staging \
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
}
