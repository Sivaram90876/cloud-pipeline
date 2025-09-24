pipeline {
    agent any

    environment {
        REGISTRY = "sivaram9087"
        IMAGE = "cloud-pipeline"
        KUBE_NAMESPACE = "default"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sivaram90876/cloud-pipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $REGISTRY/$IMAGE:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub_credentials', url: '']) {
                    sh "docker push $REGISTRY/$IMAGE:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                        kubectl set image deployment/cloud-pipeline cloud-pipeline=$REGISTRY/$IMAGE:${BUILD_NUMBER} -n $KUBE_NAMESPACE || \
                        kubectl apply -f deployment.yaml -n $KUBE_NAMESPACE
                        kubectl apply -f service.yaml -n $KUBE_NAMESPACE
                    """
                }
            }
        }
    }
}
