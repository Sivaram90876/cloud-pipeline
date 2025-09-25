pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - /busybox/sh
    args:
    - -c
    - sleep 999999
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  volumes:
  - name: docker-config
    secret:
      secretName: regcred
"""
        }
    }

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
                kubectl set image deployment/cloud-pipeline \
                  cloud-pipeline=docker.io/$IMAGE_NAME:${BUILD_NUMBER} \
                  -n default || true

                kubectl apply -f k8s/service.yaml
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
