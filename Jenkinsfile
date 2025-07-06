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

        stage('Promote a Production') {
            when {
                branch 'master'
            }
            steps {
                echo "Promoviendo a producción..."
                script {
                    withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                            echo "Haciendo merge de develop a master..."
                            git config user.name "danielcch"
                            git config user.email "daniel.camacho215@comunidadunir.net"
                            git reset --hard
                            git clean -fd
                            git checkout master
                            git pull origin master
                            git merge --strategy=recursive -X theirs develop || true
                            git checkout --theirs Jenkinsfile || true
                            git add Jenkinsfile
                            git diff --cached --quiet || git commit -m "Mergeo de develop a master [ci skip]"
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/danielcch/todo-list-aws.git HEAD:master
                        '''
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
        success {
            echo 'Despliegue en producción completado correctamente.'
        }
        failure {
            echo 'Fallo en el despliegue de producción.'
        }
    }
}
