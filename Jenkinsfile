pipeline {
    agent { label 'agente1' }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/danielcch/todo-list-aws.git',
                    credentialsId: 'GitHub_token'
            }
        }
        stage('Debug') {
            steps {
                sh 'pwd && ls -R'
            }
        }
        stage('Unit Tests') {
            steps {
 
                sh '''
                    export PYTHONPATH=$PYTHONPATH:$(pwd)
                    pytest --junitxml=unit-results.xml ./test/unit
                '''
                
                junit './unit-results.xml'
            }
        }
    }
}