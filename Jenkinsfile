pipeline {
    agent { label 'agente1' }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/danielcch/todo-list-aws.git',
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
                    pytest --cov=src --cov-report=term --cov-report=xml --cov-report=html --junitxml=unit-results.xml ./test/unit/TestToDo.py
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
                    python3 -m coverage xml
                    python3 -m coverage html
                '''
                archiveArtifacts artifacts: 'htmlcov/**/*', fingerprint: true
                publishHTML(target: [
                    reportDir: 'htmlcov',
                    reportFiles: 'index.html',
                    reportName: 'Informe de Cobertura',
                    keepAll: true
                ])
            }
        }
    
    }
}
