pipeline {
    agent any
   
    environment {
        IMAGE_NAME = "harlock12/netology-nginx"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    
    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }


        stage("get tag") {
            steps {
                script {
                    latestTag = sh(returnStdout: true, script: "git describe --tags").trim()
                }
                echo "latestTag=$latestTag"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker build -t harlock12/netology-nginx:${latestTag} .
                }
            }
        }
        
        stage('Push to Docker Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        export KUBECONFIG=$KUBECONFIG
                        # Патчим манифест или используем переменные, если нужно
                        kubectl apply -f k8s/deployment.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Успешно задеплоено: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Ошибка во время сборки или деплоя"
     
        }
    }
}
