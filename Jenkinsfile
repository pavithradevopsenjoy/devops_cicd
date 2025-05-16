pipeline {
    agent any

    environment {
        IMAGE_NAME = "pavithradocker94/abstergo-website:${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}")
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                            docker.image("${IMAGE_NAME}").push()
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-docker-desktop', variable: 'KUBECONFIG')]) {
                    script {
                        bat """
                            echo Using kubeconfig from %KUBECONFIG%
                            kubectl config get-contexts
                            kubectl set image deployment/abstergo-deployment abstergo-container=${IMAGE_NAME}
                        """
                    }
                }
            }
        }
    }
}
