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
                // Descargar el proyecto desde tu repositorio propio
                checkout scm
            }
        }
        
        stage('Setup Python Environment') {
            steps {
                echo 'Configurando entorno Python...'
                sh '''
                    python3 -m pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install pylint
                '''
            }
        }
        
        stage('Pylint Analysis') {
            steps {
                echo 'Ejecutando análisis con pylint...'
                script {
                    // Ejecutar pylint sobre todos los archivos .py del proyecto
                    // --exit-zero evita que falle el build por warnings de pylint
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
                    // Archivar el reporte de pylint
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
                    # Detener proceso anterior si existe
                    pkill -f "python3 manage.py runserver" || true
                    
                    # Iniciar servidor en background
                    nohup python3 manage.py runserver 0.0.0.0:8000 > django.log 2>&1 &
                    
                    # Esperar a que el servidor inicie
                    sleep 5
                    
                    # Verificar que el servidor está corriendo
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
