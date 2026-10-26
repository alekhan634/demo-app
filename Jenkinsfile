pipeline {
    agent any
    options { timestamps() }
    environment { 
        SCANNER_HOME = tool 'sonar-scanner' 
    }
    stages {
        stage('Test') {
            steps {
                sh 'python3 -m pytest --junitxml=result.xml --cov=app --cov-report=xml:coverage.xml || true'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        docker run --rm \
                          --network container:sonarqube-sonarqube-1 \
                          -v "$WORKSPACE:/usr/src" \
                          -w /usr/src \
                          sonarsource/sonar-scanner-cli \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    post {
        always { 
            junit 'result.xml' 
        }
        success { echo 'Сборка прошла успешно, Quality Gate пройден' }
        failure { echo 'Сборка завершилась с ошибкой' }
    }
}
