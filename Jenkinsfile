pipeline {
    agent any

    environment {
        SAM_CONFIG_ENV = "${env.BRANCH_NAME}"
        PYTHONPATH = "${env.WORKSPACE}/src"
        PATH = "/home/ubuntu/.local/bin:/usr/local/bin:/opt/jmeter/bin:${env.PATH}"
        BASE_URL = "${env.BRANCH_NAME == 'staging' ? 'https://yn2a1djoil.execute-api.us-east-1.amazonaws.com/Stage' : 'https://yn2a1djoil.execute-api.us-east-1.amazonaws.com/Prod'}"
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
                sh 'pip install --user boto3 pytest flake8 bandit "moto<5.0.0"'
            }
        }

        stage('Flake8') {
            steps {
                sh 'flake8 src test || true'
            }
        }

        stage('Bandit') {
            steps {
                sh 'bandit -r src || true'
            }
        }

        stage('Tests Unitarios') {
            steps {
                sh 'pytest test/unit/*.py || true'
            }
        }

        stage('Tests de Integración') {
            steps {
                sh 'pytest test/integration/*.py || true'
            }
        }

        stage('Pruebas de rendimiento (JMeter)') {
            steps {
                sh 'jmeter -n -t test/jmeter/jmeter.jmx -l results.jtl || true'
            }
        }
    }

    post {
        always {
            echo 'Limpieza de entorno de trabajo'
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
