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
        always {
            cleanWs()
        }
    }
}
