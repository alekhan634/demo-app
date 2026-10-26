pipeline {
    agent any
    options { timestamps() }
    environment { 
        SCANNER_HOME = tool 'sonar-scanner' 
    }
    stages {
        stage('Setup') {
            steps {
                sh 'python3 -m pip install --user pytest pytest-cov requests pyyaml'
            }
        }
        stage('Test') {
            steps {
                sh 'python3 -m pytest --junitxml=result.xml --cov=app --cov-report=xml:coverage.xml'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
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
        always { junit 'result.xml' }
        success { echo 'Сборка прошла успешно, Quality Gate пройден' }
        failure { echo 'Сборка завершилась с ошибкой' }
    }
}
