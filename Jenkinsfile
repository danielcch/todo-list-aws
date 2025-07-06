pipeline {
    agent { label 'agente1' }
    stages {
        stage('Limpiando Workspace') {
            steps {
                echo 'Cleaning workspace...'
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

        stage('Etapa test unitario') {
            environment {
                DYNAMODB_TABLE = 'todoTableTest'
            }
            steps {
                echo 'Lanzando test unitarios...'
                sh '''
                    export PYTHONPATH=$PYTHONPATH:$(pwd)
                    pytest --cov=src --cov-report=term --cov-report=xml --cov-report=html --junitxml=unit-results.xml ./test/unit/Test*.py
                '''
                junit '**/unit-results.xml'
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/unit-results.xml', fingerprint: true
                }
                failure {
                    echo 'Unit tests failed.'
                }
            }
        }

        stage('Etapa cobertura') {
            environment {
                DYNAMODB_TABLE = 'todoTableTest'
            }
            steps {
                echo 'Generando cobertura...'
                sh 'python3 -m coverage html'
                publishHTML(target: [
                    reportDir: 'htmlcov',
                    reportFiles: 'index.html',
                    reportName: 'Informe de Cobertura',
                    keepAll: true
                ])
                cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,40', lineCoverageTargets: '100,0,40'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'coverage.xml', fingerprint: true
                }
            }
        }
        stage('Etapa analisis estatico') {
            steps {
                echo 'Analisis codigo estatico (flake8)...'
                sh '''
                    python3 -m flake8 --exit-zero --format=pylint src > flake8.out || echo "No se encontraron problemas" > flake8.out
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                qualityGates: [[threshold:10, type: 'TOTAL', unstable: true], 
                [threshold: 11, type: 'TOTAL', unstable: false]]
            }
            post {
                always {
                    archiveArtifacts artifacts: 'flake8.out', fingerprint: true
                }
            }
        }

        stage('Etapa analisis seguridad') {
            steps {
                echo 'Lanzando escaneo con bandit...'
                sh '''
                    python3 -m bandit -r src -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}" || true
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], 
                qualityGates: [[threshold:1, type: 'TOTAL', unstable: true], 
                [threshold: 2, type: 'TOTAL', unstable: false]]
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bandit.out', fingerprint: true
                }
            }
        }
        stage('Etapa AWS SAM Deploy') {
            steps {
                echo 'Desplegando con SAM AWS...'
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'jenkins_aws'],
                    string(credentialsId: 'aws_session_token', variable: 'AWS_SESSION_TOKEN')
                ]) {
                    sh '''
                        sam build
                        sam validate --region us-east-1
                        sam deploy \
                            --config-env staging \
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
                                    --stack-name staging-todo-list-aws \
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

        stage('Promote a Production') {
            steps {
                script {
                    // Detectar la rama actual dinámicamente
                    def branchName = sh(
                        script: 'git rev-parse --abbrev-ref HEAD',
                        returnStdout: true
                    ).trim()
                    echo "Rama actual detectada: ${branchName}"

                    if (branchName == 'develop') {
                        echo "Rama develop detectada, ejecutando Promote..."

                        withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                            sh '''
                                echo "Haciendo merge de develop en master..."

                                # Configura user Git
                                git config user.name "danielcch"
                                git config user.email "daniel.camacho215@comunidadunir.net"

                                # DESCARTA cambios locales y limpia archivos no versionados
                                git reset --hard
                                git clean -fd
                                
                                # Cambia a master
                                git checkout master

                                # Trae los últimos cambios
                                git pull origin master

                                # Merge con estrategia que prioriza master en conflictos
                                git merge --strategy=recursive -X theirs develop || true

                                # Resuelve conflictos del Jenkinsfile priorizando master
                                git checkout --theirs Jenkinsfile || true
                                git add Jenkinsfile

                                # Commit merge (solo si hay cambios)
                                git diff --cached --quiet || git commit -m "Merge develop into master [ci skip]"

                                # Push a master usando credenciales
                                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/danielcch/todo-list-aws.git HEAD:master
                            '''
                        }
                    } else {
                        echo "No estamos en la rama develop. Promote saltado."
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline terminado.'
            cleanWs()
        }
        failure {
            echo 'Fallo Pipeline.'
        }
    }
}
