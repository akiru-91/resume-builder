pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "resume-builder"
        DOCKER_TAG = "latest"
        K8S_DEPLOYMENT = "resume-deployment"
        K8S_SERVICE = "resume-service"
    }

    stages {
        stage('Clone Repository') {
            steps {
                bat 'git clone https://github.com/akiru-91/resume-builder.git'
            }
        }

        stage('Setup Python Virtual Environment') {
            steps {
                bat '''
                python -m venv venv
                call venv\\Scripts\\activate
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests with Pytest') {
            steps {
                bat '''
                call venv\\Scripts\\activate
                pytest tests/
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-password', variable: 'DOCKER_PASSWORD')]) {
                    bat '''
                    echo %DOCKER_PASSWORD% | docker login -u akiru091 --password-stdin
                    docker tag %DOCKER_IMAGE%:%DOCKER_TAG% akiru091/%DOCKER_IMAGE%:%DOCKER_TAG%
                    docker push akiru091/%DOCKER_IMAGE%:%DOCKER_TAG%
                    '''
                }
            }
        }

        stage('Start Docker Desktop & Minikube') {
            steps {
                bat '''
                start /B "" "C:\\Program Files\\Docker\\Docker\\Docker Desktop.exe"
                timeout /t 30
                minikube start
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '''
                kubectl get pods
                kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}
