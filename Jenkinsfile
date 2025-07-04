pipeline {
    agent { label 'agente1' }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'develop',
                    url: 'https://github.com/danielcch/todo-list-aws.git',
                    credentialsId: 'GitHub_token'
            }
        }
    }
}