pipeline {
    agent any

    environment {
        IMAGE_NAME = "sivaram9087/cloud-pipeline"
        NAMESPACE = "default"   // change to jenkins if that's where you deploy
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
                echo '📦 Applying Deployment & Service...'
                kubectl apply -f k8s/deployment.yaml -n $NAMESPACE
                kubectl apply -f k8s/service.yaml -n $NAMESPACE

                echo '🔄 Updating Deployment image...'
                kubectl set image deployment/cloud-pipeline \
                  cloud-pipeline=docker.io/$IMAGE_NAME:${BUILD_NUMBER} \
                  -n $NAMESPACE || true
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
