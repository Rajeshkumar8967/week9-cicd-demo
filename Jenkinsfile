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
                bat '"%PYTHON%" -m pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                echo "Running automated tests..."
                bat '"%PYTHON%" -m pytest -v'
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
                    if %ERRORLEVEL% NEQ 0 exit /b 1
                '''
            }
        }

        stage('Docker Validation') {
            steps {
                echo "Validating Docker image..."
                bat '''
                    docker image inspect %DOCKER_IMAGE%:%BUILD_NUMBER%
                    if %ERRORLEVEL% NEQ 0 exit /b 1
                '''
            }
        }

        stage('Credential Diagnostic') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'dockerhub-pat-2',
                        variable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    powershell '''
                        $bytes = [System.Text.Encoding]::UTF8.GetBytes($env:DOCKER_PASSWORD)
                        $sha = New-Object System.Security.Cryptography.SHA256Managed
                        $hashBytes = $sha.ComputeHash($bytes)
                        $hash = ([BitConverter]::ToString($hashBytes)).Replace('-', '').ToLower()
                        Write-Host "Docker credential length: $($env:DOCKER_PASSWORD.Length)"
                        Write-Host "Docker credential SHA256: $hash"
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                echo "Authenticating with Docker Hub and pushing image..."
                withCredentials([
                    string(
                        credentialsId: 'dockerhub-pat-2',
                        variable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    powershell '''
                        docker logout docker.io 2>$null
                        $env:DOCKER_PASSWORD | & docker login docker.io -u rajeshkumar357 --password-stdin
                        if ($LASTEXITCODE -ne 0) {
                            Write-Host "JENKINS AUTHENTICATION FAILED"
                            exit 1
                        }

                        Write-Host "JENKINS AUTHENTICATION SUCCESSFUL"
                        & docker push "$env:DOCKER_IMAGE`:$env:BUILD_NUMBER"
                        if ($LASTEXITCODE -ne 0) {
                            Write-Host "DOCKER PUSH FAILED"
                            exit 1
                        }

                        Write-Host "DOCKER PUSH SUCCESSFUL"
                        & docker logout docker.io
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "======================================"
            echo "WEEK 9 CI/CD PIPELINE SUCCESS"
            echo "Application: ${env.APP_NAME}"
            echo "Docker Image: ${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}"
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