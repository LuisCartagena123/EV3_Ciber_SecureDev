pipeline {
    agent any

    stages {
        stage('Construccion (Build)') {
            steps {
                echo 'Construyendo imagen Docker de SecureDev...'
                sh 'docker build -t securedev-app:latest .'
            }
        }

        stage('Pruebas de Seguridad (OWASP ZAP)') {
            steps {
                echo 'Levantando contenedor temporal y ejecutando OWASP ZAP...'
                sh '''
                    docker rm -f app-zap-test || true
                    docker run -d --name app-zap-test -p 5050:5000 securedev-app:latest
                    sleep 5
                    mkdir -p zap-report
                    docker run --rm -v "$(pwd)/zap-report:/zap/wrk/:rw" ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://host.docker.internal:5050 -r zap_report.html || true
                    docker rm -f app-zap-test
                '''
            }
        }

        stage('Despliegue (Deploy)') {
            steps {
                echo 'Desplegando aplicacion en produccion con Docker Compose...'
                sh 'docker compose down || true'
                sh 'docker compose up -d'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'zap-report/zap_report.html', allowEmptyArchive: true
        }
    }
}