pipeline {
    agent any

    stages {
        stage('Construccion (Build)') {
            steps {
                echo 'Construyendo imagen Docker de SecureDev...'
                bat 'docker build -t securedev-app:latest .'
            }
        }

        stage('Pruebas de Seguridad (OWASP ZAP)') {
            steps {
                echo 'Levantando contenedor temporal y ejecutando OWASP ZAP...'
                bat '''
                    docker rm -f app-zap-test 2>nul || exit /b 0
                    docker run -d --name app-zap-test -p 5050:5000 securedev-app:latest
                    timeout /t 5 /nobreak > nul
                    if not exist zap-report mkdir zap-report
                    docker run --rm -v "%WORKSPACE%\\zap-report:/zap/wrk/:rw" ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://host.docker.internal:5050 -r zap_report.html || exit /b 0
                    docker rm -f app-zap-test
                '''
            }
        }

        stage('Despliegue (Deploy)') {
            steps {
                echo 'Desplegando aplicacion en produccion con Docker Compose...'
                bat 'docker compose down 2>nul || exit /b 0'
                bat 'docker compose up -d'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'zap-report/zap_report.html', allowEmptyArchive: true
        }
    }
}