pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')  // Stored in Jenkins
        DOCKER_IMAGE = "prateekdocker/new-po:${BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/prateekniet/azure-demo-cicd.git',branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  cd vote
                  docker build -t ${DOCKER_IMAGE} .
                """
            }
        }

        stage('DockerHub Login & Push') {
            steps {
                sh """
                  echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                  docker push ${DOCKER_IMAGE}
                """
            }
        }

        stage('Deploy to Kind Cluster') {
            steps {
                sh """
                  # Make sure kubectl is configured to point to kind cluster
                  kubectl config use-context kind-netpol-demo

                  # Update image in deployment.yaml (optional)
                  # For simplicity, let's patch the image directly
                  kubectl set image deployment/vote vote-demo=${DOCKER_IMAGE} --record || \
                  kubectl apply -f k8s-specifications/vote-deployment.yaml
                  kubectl apply -f k8s-specifications/vote-service.yaml

                  kubectl rollout status deployment/vote
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful! 🎉"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
