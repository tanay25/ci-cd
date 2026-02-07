pipeline {
    agent any

    environment {
        IMAGE_NAME = "tanay25/flask-devsecops"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/tanay25/ci-cd.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                pip install pytest bandit safety
                '''
            }
        }

        stage('Unit Testing') {
            steps {
                sh '''
                . venv/bin/activate
                pytest
                '''
            }
        }

        stage('SAST - Static Code Analysis (Bandit)') {
            steps {
                sh '''
                . venv/bin/activate
                bandit -r app.py
                '''
            }
        }

        stage('Dependency Vulnerability Scan (Safety)') {
            steps {
                sh '''
                . venv/bin/activate
                safety check || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Container Security Scan (Trivy)') {
            steps {
                sh '''
                trivy image $IMAGE_NAME:$IMAGE_TAG || true
                '''
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                docker rm -f flask-app || true
                docker run -d -p 5000:5000 --name flask-app $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD + Security Pipeline Successful"
        }
        failure {
            echo "Pipeline Failed – Check Logs"
        }
    }
}

