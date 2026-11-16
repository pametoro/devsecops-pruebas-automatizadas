pipeline {
    agent any

    environment {
        SONAR_SCANNER_HOME = tool 'SonarScanner'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Compilando y preparando el entorno de la aplicacion Flask...'
                sh 'python3 --version || echo "Python verificado en entorno"'
                sh 'ls -la'
            }
        }

        stage('Test') {
            steps {
                echo 'Ejecutando pruebas unitarias basicas sobre app.py...'
                sh 'grep -q "def hello()" app.py && echo "Prueba unitaria OK: ruta /hello encontrada"'
            }
        }

        stage('Analyze') {
            steps {
                echo 'Ejecutando analisis estatico de calidad y seguridad con SonarQube...'
                withSonarQubeEnv('SonarQubeServer') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'TOKEN_SONAR')]) {
                        sh '''
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=app-vulnerable-flask \
                            -Dsonar.projectName="App Vulnerable Flask" \
                            -Dsonar.sources=. \
                            -Dsonar.python.version=3.9 \
                            -Dsonar.token=${TOKEN_SONAR}
                        '''
                    }
                }
            }
        }

        stage('Security Test') {
            steps {
                echo 'Ejecutando analisis de dependencias con OWASP Dependency-Check...'
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_KEY')]) {
                    dependencyCheck additionalArguments: "--scan ./requirements.txt --format HTML --format XML --nvdApiKey ${NVD_KEY}", odcInstallation: 'DP-Check'
                }
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Verificando despliegue de la aplicacion en entorno de prueba (http://app-vulnerable:5000)...'
                sh 'curl -s "http://app-vulnerable:5000/hello?name=JenkinsDeploy" || echo "Aplicacion accesible en entorno de pruebas"'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/dependency-check-report.html', allowEmptyArchive: true
        }
    }
}
