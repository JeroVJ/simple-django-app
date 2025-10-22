pipeline {
    agent any
    
    environment {
        PYTHON_VERSION = '3.9'
        APP_DIR = 'cool_counters'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Descargando código desde el repositorio...'
                checkout scm
            }
        }
        
        stage('Setup Python Environment') {
            steps {
                echo 'Configurando entorno Python...'
                sh '''
                    python3 -m pip install --upgrade pip --break-system-packages
                    pip install -r requirements.txt --break-system-packages
                    pip install pylint --break-system-packages
                '''
            }
        }
        
        stage('Pylint Analysis') {
            steps {
                echo 'Ejecutando análisis con pylint...'
                script {
                    sh '''
                        pylint --exit-zero --output-format=text \
                               --reports=y \
                               --recursive=y \
                               ${APP_DIR}/ > pylint-report.txt || true
                        
                        echo "=== Resultados de Pylint ==="
                        cat pylint-report.txt
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'pylint-report.txt', allowEmptyArchive: true
                }
            }
        }
        
        stage('Run Migrations') {
            steps {
                echo 'Ejecutando migraciones de Django...'
                sh '''
                    cd ${APP_DIR}
                    python3 manage.py migrate --noinput
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                echo 'Ejecutando tests de Django...'
                sh '''
                    cd ${APP_DIR}
                    python3 manage.py test
                '''
            }
        }
        
        stage('Deploy Application') {
            steps {
                echo 'Desplegando aplicación Django...'
                sh '''
                    cd ${APP_DIR}
                    pkill -f "python3 manage.py runserver" || true
                    
                    nohup python3 manage.py runserver 0.0.0.0:8000 > django.log 2>&1 &
                    
                    sleep 5
                    
                    if curl -f http://localhost:8000/ ; then
                        echo "Aplicación desplegada exitosamente"
                    else
                        echo "Error al desplegar la aplicación"
                        exit 1
                    fi
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completado exitosamente!'
        }
        failure {
            echo 'Pipeline falló. Revise los logs.'
        }
        always {
            echo 'Limpiando workspace...'
            cleanWs()
        }
    }
}
