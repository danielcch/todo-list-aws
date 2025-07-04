pipeline {
    agent { label 'agente1' }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/danielcch/todo-list-aws.git',
                    credentialsId: 'GitHub_token'
            }
        }
        stage('Unit Tests') {
            steps {
                dir('todo-list-aws') {
                    sh '''
                            export PYTHONPATH=$PYTHONPATH:$(pwd)
                            pytest --junitxml=unit-results.xml test/unit/
                        '''
                }
            }
        }
    }
}