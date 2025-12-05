pipeline {
    agent none

    tools {
        sonarQubeScanner 'SonarScanner'   // must match Global Tool Configuration
    }

    stages {

        stage('Checkout Code') {
            agent { label 'built-in' }
            steps {
                checkout scm
                stash includes: '**', name: 'source-code'
            }
        }

        stage('SonarQube Scan') {
            agent { label 'built-in' }
            steps {
                unstash 'source-code'
                withSonarQubeEnv('sonarqube-server') {   // must match Configure System
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=django-sample \
                          -Dsonar.sources=. \
                          -Dsonar.python.version=3.14 \
                          -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }

        stage("Wait for Quality Gate") {
            agent { label 'built-in' }
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Lint Code (PyLint)') {
            agent { label 'pynode' }
            steps {
                unstash 'source-code'
                sh '''
                    pylint --rcfile=.pylintrc greet/ sample/ > pylint-report.txt || true
                '''
            }
            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        tool: pylint(pattern: 'pylint-report.txt')
                    )
                }
            }
        }

        stage('Run Tests (PyTest)') {
            agent { label 'pynode' }
            steps {
                unstash 'source-code'
                sh '''
                    pytest --junitxml=pytest-results.xml
                '''
            }
            post {
                always {
                    junit 'pytest-results.xml'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
