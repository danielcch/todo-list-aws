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
 
                sh '''
                    export PYTHONPATH=$PYTHONPATH:$(pwd)
                    pip uninstall --break-system-packages boto3
                    pip uninstall --break-system-packages moto
                    pip install --break-system-packages -r ./src/requirements.txt
                    pytest --junitxml=unit-results.xml ./test/unit/TestToDo.py
                '''
                
                junit './unit-results.xml'
            }
        }
    }
}