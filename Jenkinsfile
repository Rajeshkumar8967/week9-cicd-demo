pipeline {
    agent any

    environment {
        APP_NAME = "week9-cicd-demo"

        // IMPORTANT:
        // This must match your actual Docker Hub username/repository.
        DOCKER_IMAGE = "rajeshkumar357/week9-cicd-demo"

        // Python installed on your Windows machine
        PYTHON = "C:\\Users\\91703\\AppData\\Local\\Programs\\Python\\Python314\\python.exe"
    }

    stages {

        // ==============================
        // 1. CHECKOUT
        // ==============================
        stage('Checkout') {
            steps {
                echo "Checking out source code from GitHub..."

                checkout scm
            }
        }

        // ==============================
        // 2. BUILD
        // ==============================
        stage('Build') {
            steps {
                echo "Installing Python dependencies..."

                bat '''
                    "%PYTHON%" -m pip install -r requirements.txt
                '''
            }
        }

        // ==============================
        // 3. TEST
        // ==============================
        stage('Test') {
            steps {
                echo "Running automated tests..."

                bat '''
                    "%PYTHON%" -m pytest -v
                '''
            }
        }

        // ==============================
        // 4. PACKAGE
        // ==============================
        stage('Package') {
            steps {
                echo "Creating application package..."

                bat '''
                    if exist package rmdir /s /q package
                    mkdir package

                    copy app.py package\\
                    copy requirements.txt package\\
                '''

                archiveArtifacts artifacts: 'package/**',
                                 fingerprint: true
            }
        }

        // ==============================
        // 5. DOCKER BUILD
        // ==============================
        stage('Docker Build') {
            steps {
                echo "Building Docker image..."

                bat '''
                    docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .
                '''
            }
        }

        // ==============================
        // 6. DOCKER VALIDATION
        // ==============================
        stage('Docker Validation') {
            steps {
                echo "Validating Docker image..."

                bat '''
                    docker image inspect %DOCKER_IMAGE%:%BUILD_NUMBER%
                '''
            }
        }

        // ==============================
        // 7. DOCKER PUSH
        // ==============================
        stage('Docker Push') {
            steps {

                echo "Logging in to Docker Hub and pushing image..."

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    bat '''
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin

                        if %ERRORLEVEL% NEQ 0 (
                            echo Docker login FAILED
                            exit /b 1
                        )

                        docker push %DOCKER_IMAGE%:%BUILD_NUMBER%

                        if %ERRORLEVEL% NEQ 0 (
                            echo Docker push FAILED
                            exit /b 1
                        )

                        docker logout
                    '''
                }
            }
        }
    }

    // ==============================
    // POST BUILD ACTIONS
    // ==============================
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