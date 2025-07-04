pipeline {
    agent { label 'agente1' }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/danielcch/new-todo-list-aws.git',
                    credentialsId: 'GitHub_token'
            }
        }
    }
}