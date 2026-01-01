pipeline {
    agent none

    options {
        skipDefaultCheckout(true)
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
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        /opt/sonar-scanner/bin/sonar-scanner \
                          -Dsonar.projectKey=django-sample-dev \
                          -Dsonar.sources=. \
                          -Dsonar.python.version=3.10 \
                          -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }

        stage('Wait for Quality Gate') {
                  agent { label 'built-in' }
                 steps {
                         echo 'Waiting for SonarQube to process the reports...'
                        sleep(time: 45, unit: 'SECONDS')
                     timeout(time: 15, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
        stage('Lint Code (PyLint)') {
            agent { label 'pynode' }
            steps {
                unstash 'source-code'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt pylint

                    pylint greet/ sample/ || true
                '''
            }
        }

        stage('Run Tests (PyTest)') {
            agent { label 'pynode' }
            steps {
                unstash 'source-code'
                sh '''
                    . venv/bin/activate
                    pip install pytest
                    pytest --junitxml=pytest-results.xml
                '''
            }
            post {
                always {
                    junit 'pytest-results.xml'
                }
            }
        }

        stage('Raise PR to dev-saeed') {
            agent { label 'built-in' }
            when {
                branch 'dev-saeed'
            }
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        export GH_TOKEN=$GITHUB_TOKEN

                        gh pr create \
                          --base main \
                          --head dev-saeed \
                          --title "Auto PR: Merge dev-saeed to main" \
                          --body "Pipeline passed. Auto-generated PR."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
