pipeline {
    agent any

    environment {
        IMAGE_NAME = "salhianis20/todo-react-app"
        TAG = "latest"
        SONARQUBE_ENV = 'SonarQube'
        SONAR_PROJECT_KEY = 'ToDo-React-App'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        
        stage('SonarQube Scan') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'SonarQube-ID', variable: 'jenkins')]) {
                        withSonarQubeEnv("${env.SONARQUBE_ENV}") {
                            sh """
                                sonar-scanner \
                                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                -Dsonar.sources=. \
                                -Dsonar.token=${jenkins}
                            """
                        }
                    }
                }
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
                    sh "trivy image $IMAGE_NAME:$TAG"
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

