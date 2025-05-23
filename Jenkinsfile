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
        stage('Debug Kubectl') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-docker-desktop', variable: 'KUBECONFIG')]) {
      bat """
        echo Kubeconfig file path: %KUBECONFIG%
        kubectl config view
        kubectl config current-context
        kubectl config get-contexts
        kubectl get namespaces
        kubectl get deployments --all-namespaces
      """
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
          kubectl get deployments -n default
          kubectl set image deployment/abstergo-deployment abstergo-container=${IMAGE_NAME} -n default
          kubectl rollout status deployment/abstergo-deployment -n default
          kubectl get pods -l app=abstergo -n default
        """
                    }
                }
            }
        }
    }
}
