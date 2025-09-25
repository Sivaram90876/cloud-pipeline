pipeline {
    agent any

    environment {
        IMAGE_NAME = "sivaram9087/cloud-pipeline"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Sivaram90876/Pipeline-test.git'
            }
        }

        stage('Build & Push with Kaniko') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                      --context ${WORKSPACE} \
                      --dockerfile ${WORKSPACE}/dockerfile \
                      --destination=$IMAGE_NAME:${BUILD_NUMBER} \
                      --destination=$IMAGE_NAME:latest \
                      --oci-layout-path=/kaniko/oci \
                      --verbosity=info
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                kubectl apply -f k8s/ingress.yaml
                """
            }
        }
    }

    post {
        success {
            echo "✅ Successfully built & pushed image to DockerHub!"
            echo "🚀 Deployed to EKS!"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
