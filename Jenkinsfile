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
                        pytest --junitxml=unit-results.xml test/unit/
                    '''
                }
            }

            stage('Coverage') {
                steps {
                    sh '''
                        export PYTHONPATH=$PYTHONPATH:$(pwd)
                        python3 -m coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                        python3 -m coverage xml
                    '''
                }
            }
    
    }
}
