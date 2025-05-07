pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "harlock12/netology-app"
        REGISTRY_CREDENTIALS = 'e028188b-1ea3-490d-b895-5f0bdccf4841'  // ID в Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_TAG = sh(
                        script: "git describe --tags --exact-match || true",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def imageTag = env.GIT_TAG ?: "latest"
                    sh """
                      docker build -t $DOCKER_IMAGE:$imageTag .
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push $DOCKER_IMAGE:$imageTag
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { return env.GIT_TAG }
            }
            steps {
                sh """
                  kubectl set image deployment/nginx-test nginx=$DOCKER_IMAGE:$GIT_TAG -n jenkins
                """
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed!"
        }
    }
}
