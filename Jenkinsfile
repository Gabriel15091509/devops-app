pipeline {
    agent any

    environment {
        IMAGE_NAME = "devops-app"
        DOCKERHUB_CREDENTIALS = "dockerhub-credentials"
        KUBECONFIG = "C:\\ProgramData\\Jenkins\\.kube\\config"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('devops-app') {
                    bat 'docker build -t %IMAGE_NAME%:latest .'
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: DOCKERHUB_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %IMAGE_NAME%:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('devops-app') {
                    bat '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    '''
                }
            }
        }

        stage('Get App URL') {
            steps {
                bat '''
                echo ==============================
                echo URL DE L'APPLICATION :
                minikube service devops-app-service --url
                echo ==============================
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD COMPLET : Docker + Kubernetes OK"
        }
        failure {
            echo "❌ Pipeline échoué — vérifie les logs"
        }
    }
}
