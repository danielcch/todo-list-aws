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
                DYNAMODB_TABLE = 'todoTableTest' // Nombre de la tabla DynamoDB para pruebas unitarias
            }   
            steps {
 
                sh '''
                    export PYTHONPATH=$PYTHONPATH:$(pwd)
                    pytest --cov=src --cov-report=xml --cov-report=html --junitxml=unit-results.xml ./test/unit/TestToDo.py
                '''//requests.txt las librerias boto3nmoto requests las tienen los nodo pq se dijo de no hacer entonrno virtual
                
                junit '**/unit-results.xml' // Publica los resultados de las pruebas unitarias
            }
        }

        stage('Coverage Report') {
            steps {
                sh '''
                    coverage report
                    coverage xml
                '''
            }
        }
    }
}
