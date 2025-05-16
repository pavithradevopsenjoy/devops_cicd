pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("pavithradocker94/abstergo-website:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                            docker.image("pavithradocker94/abstergo-website:${env.BUILD_NUMBER}").push()
                        }
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
    steps {
        script {
            if (isUnix()) {
                sh 'kubectl apply -f deployment.yaml'
            } else {
                bat 'kubectl apply -f deployment.yaml'
            }
        }
    }
}

