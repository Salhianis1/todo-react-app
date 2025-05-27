pipeline {
    agent any

    environment {
        IMAGE_NAME = "salhianis20/todo-react-app"
        TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME:$TAG ."
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh "trivy image --exit-code 1 --severity CRITICAL,HIGH $IMAGE_NAME:$TAG"
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-id',
                                                 usernameVariable: 'DOCKERHUB_USER',
                                                 passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh "echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "docker push $IMAGE_NAME:$TAG"
                }
            }
        }
    }

    post {
        always {
            sh "docker logout"
        }
        success {
            echo "Build, scan, and push completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
    }
}

