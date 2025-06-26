pipeline {
    agent any

    environment {
        SAM_CONFIG_ENV = "${env.BRANCH_NAME}"
        PYTHONPATH = "${env.WORKSPACE}"
        PATH = "/home/ubuntu/.local/bin:/usr/local/bin:/opt/jmeter/bin:${env.PATH}"
        BASE_URL = "${env.BRANCH_NAME == 'staging' ? 'https://yn2a1djoil.execute-api.us-east-1.amazonaws.com/Stage' : 'https://yn2a1djoil.execute-api.us-east-1.amazonaws.com/Prod'}"
        DYNAMODB_TABLE = 'ToDoTable'
    }

    stages {
        stage('Mostrar configuración') {
            steps {
                echo "Desplegando usando entorno: ${SAM_CONFIG_ENV}"
                echo "URL base usada en tests: ${BASE_URL}"
                sh 'cat samconfig.toml'
            }
        }

        stage('Desplegar con SAM') {
            steps {
                sh """
                    sam deploy \
                    --config-env ${SAM_CONFIG_ENV} \
                    --no-confirm-changeset \
                    --no-fail-on-empty-changeset
                """
            }
        }

        stage('Instalar dependencias') {
            steps {
                sh 'pip install --user boto3 pytest flake8 bandit moto'
            }
        }

        stage('Flake8') {
            steps {
                sh 'flake8 src test > flake8-report.txt || true'
            }
        }

        stage('Bandit') {
            steps {
                sh 'bandit -r src -f html -o bandit-report.html || true'
            }
        }

        stage('Tests Unitarios') {
            steps {
                sh 'pytest test/unit/TestToDo.py --junitxml=unit-tests.xml || true'
            }
        }

        stage('Tests de Integración') {
            steps {
                sh 'pytest test/integration --junitxml=integration-tests.xml || true'
            }
        }

        stage('Pruebas de rendimiento (JMeter)') {
            steps {
                sh 'jmeter -n -t test/jmeter/jmeter.jmx -l results.jtl -e -o jmeter-report || true'
            }
        }
    }

    post {
        always {
            echo 'Limpieza de entorno de trabajo'
            archiveArtifacts artifacts: 'flake8-report.txt', allowEmptyArchive: true
            archiveArtifacts artifacts: 'bandit-report.html', allowEmptyArchive: true
            archiveArtifacts artifacts: 'unit-tests.xml', allowEmptyArchive: true
            archiveArtifacts artifacts: 'integration-tests.xml', allowEmptyArchive: true
            archiveArtifacts artifacts: 'results.jtl', allowEmptyArchive: true
            archiveArtifacts artifacts: 'jmeter-report/**', allowEmptyArchive: true
            junit 'unit-tests.xml'
            junit 'integration-tests.xml'
            cleanWs()
        }
        success {
            echo "Despliegue y pruebas completadas correctamente en ${SAM_CONFIG_ENV}"
        }
        failure {
            echo "Error durante el pipeline en ${SAM_CONFIG_ENV}"
        }
    }
}
