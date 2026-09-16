pipeline {
    agent any

    environment {
        APP_NAME = "week9-cicd-demo"
        DOCKER_IMAGE = "YOUR_DOCKERHUB_USERNAME/week9-cicd-demo"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat '''
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                    python -m pytest -v
                '''
            }
        }

        stage('Package') {
            steps {
                bat '''
                    if not exist package mkdir package
                    copy app.py package\\
                    copy requirements.txt package\\
                '''

                archiveArtifacts artifacts: 'package/**', fingerprint: true
            }
        }

        stage('Docker Build') {
            steps {
                bat '''
                    docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .
                '''
            }
        }

        stage('Docker Validation') {
            steps {
                bat '''
                    docker image inspect %DOCKER_IMAGE%:%BUILD_NUMBER%
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat '''
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                        docker push %DOCKER_IMAGE%:%BUILD_NUMBER%
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Week 9 CI/CD Weekly Task completed successfully.'
        }

        failure {
            echo 'Week 9 CI/CD pipeline failed.'
        }
    }
}