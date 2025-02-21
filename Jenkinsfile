pipeline {
    agent any
    environment {
        PYTHON_PATH = "/c/Users/akhil/AppData/Local/Programs/Python/Python3X/python.exe"
    }
    stages {
        stage('Clone Repository') {
            steps {
                sh """
                    rm -rf resume-builder || true  # Remove old repo if exists
                    git clone https://github.com/akiru-91/resume-builder.git
                """
            }
        }

        stage('Setup Python Virtual Env') {
            steps {
                sh """
                    ${PYTHON_PATH} -m venv venv
                    source venv/Scripts/activate
                    pip install -r requirements.txt
                """
            }
        }

        stage('Run Tests') {
            steps {
                sh """
                    source venv/Scripts/activate
                    pytest tests/
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t resume-builder:latest .
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    echo "\$DOCKER_PASSWORD" | docker login -u "\$DOCKER_USERNAME" --password-stdin
                    docker tag resume-builder:latest akiru091/resume-builder:latest
                    docker push akiru091/resume-builder:latest
                """
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh """
                    minikube start
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
}
