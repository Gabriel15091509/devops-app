pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "gabriellova"
        IMAGE_NAME = "devops-app"
        FULL_IMAGE_NAME = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
        DOCKERHUB_CRED = "dockerhub-credentials"
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
                    bat "docker build -t %FULL_IMAGE_NAME% ."
                }
            }
        }

       stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKERHUB_CRED,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    docker logout
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    """
                }
            }
        }


        stage('Push Docker Image') {
            steps {
                bat "docker push %FULL_IMAGE_NAME%"
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
            echo "✅ CI/CD COMPLET : DockerHub + Kubernetes OK 🚀"
        }
        failure {
            echo "❌ Pipeline échoué — vérifie les logs Jenkins"
        }
    }
}
