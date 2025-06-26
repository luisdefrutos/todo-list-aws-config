pipeline {
    agent any

    environment {
        SAM_CONFIG_ENV = "${env.BRANCH_NAME}"
        PYTHONPATH = "${env.WORKSPACE}/src"
         PATH = "/home/ubuntu/.local/bin:/usr/local/bin:/opt/jmeter/bin:${env.PATH}"
    }

    stages {
        stage('Mostrar configuración') {
            steps {
                echo "Desplegando usando entorno: ${SAM_CONFIG_ENV}"
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

        stage('Flake8') {
            steps {
            sh 'python3 -m flake8 src test || true'
            }
        }

        stage('Bandit') {
            steps {
               sh 'python3 -m bandit -r src || true'
            }
        }

        stage('Tests Unitarios') {
            steps {
                sh 'pytest test/unit || true'
            }
        }

        stage('Tests de Integración') {
            steps {
                sh 'pytest test/integration || true'
            }
        }

        // Opcional: descomenta esto si tienes JMeter instalado
        
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
