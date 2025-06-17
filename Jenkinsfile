pipeline {
    agent any

    environment {
        SAM_CONFIG_ENV = "${env.BRANCH_NAME}"
    }

    stages {
        stage('Mostrar configuración') {
            steps {
                echo " Desplegando usando entorno: ${SAM_CONFIG_ENV}"
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
    }

    post {
        success {
            echo "Despliegue completado correctamente en ${SAM_CONFIG_ENV}"
        }
        failure {
            echo "Error durante el despliegue en ${SAM_CONFIG_ENV}"
        }
    }
}
