pipeline {
    agent any

    environment {
        APP_NAME = "week9-cicd-demo"
        DOCKER_IMAGE = "rajeshkumar357/week9-cicd-demo"
        PYTHON = "C:\\Users\\91703\\AppData\\Local\\Programs\\Python\\Python314\\python.exe"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code from GitHub..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Installing Python dependencies..."
                bat '''
                    "%PYTHON%" -m pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo "Running automated tests..."
                bat '''
                    "%PYTHON%" -m pytest -v
                '''
            }
        }

        stage('Package') {
            steps {
                echo "Creating application package..."
                bat '''
                    if exist package rmdir /s /q package
                    mkdir package
                    copy app.py package\\
                    copy requirements.txt package\\
                '''
                archiveArtifacts artifacts: 'package/**', fingerprint: true
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image..."
                bat '''
                    docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .
                '''
            }
        }

        stage('Docker Validation') {
            steps {
                echo "Validating Docker image..."
                bat '''
                    docker image inspect %DOCKER_IMAGE%:%BUILD_NUMBER%
                '''
            }
        }

        stage('Docker Push') {
            steps {
                echo "Testing Docker Hub authentication from Jenkins..."
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat '''
                        echo Jenkins Docker username: %DOCKER_USERNAME%
                        docker logout
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                        if %ERRORLEVEL% NEQ 0 (
                            echo ======================================
                            echo JENKINS DOCKER LOGIN FAILED
                            echo ======================================
                            exit /b 1
                        )
                        echo ======================================
                        echo JENKINS DOCKER LOGIN SUCCESS
                        echo ======================================
                        docker info
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "======================================"
            echo "WEEK 9 CI/CD PIPELINE SUCCESS"
            echo "Application: %APP_NAME%"
            echo "Docker Image: %DOCKER_IMAGE%:%BUILD_NUMBER%"
            echo "======================================"
        }
        failure {
            echo "======================================"
            echo "WEEK 9 CI/CD PIPELINE FAILED"
            echo "Check the Jenkins console output."
            echo "======================================"
        }
        always {
            echo "CI/CD pipeline execution completed."
        }
    }
}