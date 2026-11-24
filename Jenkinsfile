
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Repository Validation') {
            steps {
                sh '''
                    set -eu

                    test -s login.py

                    echo "Source file exists and is not empty."
                    echo "Commit: $(git rev-parse --short HEAD)"
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'

                    withSonarQubeEnv('sonarqube-local') {

                        sh """
                            "${scannerHome}/bin/sonar-scanner" \
                              -Dsonar.projectKey=devops-project \
                              -Dsonar.projectName=DevOps-Project \
                              -Dsonar.sources=. \
                              -Dsonar.sourceEncoding=UTF-8 \
                              -Dsonar.exclusions=Jenkinsfile
                        """
                    }
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
        success {
            echo 'Pipeline completed successfully. Quality Gate passed.'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}

