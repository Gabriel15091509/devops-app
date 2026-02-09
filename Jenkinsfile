pipeline {
    agent any

    environment {
        // Nomme ici ton credentials ID créé dans Jenkins pour Docker Hub
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        IMAGE_NAME = 'gabriellova/devops-app'
        KUBE_CONFIG_CREDENTIALS = 'kubeconfig-credentials' // Credentials contenant le fichier kubeconfig
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:latest")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS,
                                                      usernameVariable: 'DOCKER_USER',
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: KUBE_CONFIG_CREDENTIALS, variable: 'KUBECONFIG_FILE')]) {
                        sh 'mkdir -p $HOME/.kube'
                        sh 'cp $KUBECONFIG_FILE $HOME/.kube/config'
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                    }
                }
            }
        }

        stage('Get App URL') {
            steps {
                script {
                    sh 'minikube service devops-app-service --url'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès !"
        }
        failure {
            echo "❌ Pipeline échoué."
        }
    }
}
