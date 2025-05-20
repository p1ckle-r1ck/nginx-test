pipeline {
    agent any
   
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
                echo " ============== start building image =================="
                dir ('.') {
                	sh """
                    docker build -t harlock12/netology-nginx:$latestTag . 
                    """
                }
            }
        }
        // environment {
        //     IMAGE_NAME = "harlock12/netology-nginx"

        // }

        stage('Push to Docker Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push harlock12/netology-nginx:${latestTag}
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
                        kubectl apply -f deployment.yml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Успешно задеплоено: harlock12/netology:${latestTag}"
        }
        failure {
            echo "❌ Ошибка во время сборки или деплоя"
     
        }
    }
}
