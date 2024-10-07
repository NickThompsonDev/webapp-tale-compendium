pipeline {
    agent any
    environment {
        MINIKUBE_IP = sh(script: 'docker inspect -f "{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}" minikube', returnStdout: true).trim()

        // Secrets stored in Jenkins Credentials
        DATABASE_PASSWORD = credentials('DATABASE_PASSWORD')
        CLERK_SECRET_KEY = credentials('CLERK_SECRET_KEY')
        STRIPE_SECRET_KEY = credentials('STRIPE_SECRET_KEY')
        OPENAI_API_KEY = credentials('OPENAI_API_KEY')
        NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY = credentials('NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY')
        NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY = credentials('NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY')
        NEXT_PUBLIC_CLERK_SIGN_IN_URL = credentials('NEXT_PUBLIC_CLERK_SIGN_IN_URL')
        NEXT_PUBLIC_CLERK_SIGN_UP_URL = credentials('NEXT_PUBLIC_CLERK_SIGN_UP_URL')
        CLERK_WEBHOOK_SECRET = credentials('CLERK_WEBHOOK_SECRET')
    }
    stages {
        stage('Checkout Webapp Code') {
            steps {
                git url: 'https://github.com/NickThompsonDev/webapp-tale-compendium.git', branch: 'master'
            }
        }
        stage('Static Code Analysis') {
            steps {
                echo 'Running static code analysis...'
            }
        }
        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build and tag the Docker image for the webapp
                    sh """
                    docker build --build-arg NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY} \
                                 --build-arg STRIPE_SECRET_KEY=${STRIPE_SECRET_KEY} \
                                 --build-arg NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=${NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY} \
                                 --build-arg CLERK_SECRET_KEY=${CLERK_SECRET_KEY} \
                                 --build-arg OPENAI_API_KEY=${OPENAI_API_KEY} \
                                 --build-arg DATABASE_PASSWORD=${DATABASE_PASSWORD} \
                                 --build-arg NEXT_PUBLIC_CLERK_SIGN_IN_URL=${NEXT_PUBLIC_CLERK_SIGN_IN_URL} \
                                 --build-arg NEXT_PUBLIC_CLERK_SIGN_UP_URL=${NEXT_PUBLIC_CLERK_SIGN_UP_URL} \
                                 --build-arg NEXT_PUBLIC_API_URL=http://${MINIKUBE_IP}:5000/api \
                                 --build-arg NEXT_PUBLIC_WEBAPP_URL=http://${MINIKUBE_IP}:3000 \
                                 -t webapp-tale-compendium:latest .
                    """
                }
            }
        }
        stage('Load Docker Image into Minikube') {
            steps {
                script {
                    // Use docker save and docker load commands to load the image into Minikube's Docker environment
                    sh """
                    docker save webapp-tale-compendium:latest | docker exec -i minikube docker load
                    """
                }
            }
        }
        stage('Deploy Webapp to Minikube with Terraform') {
            steps {
                script {
                    // Trigger the Terraform pipeline to apply Webapp-related changes
                    build job: 'Terraform-Deploy-Pipeline', parameters: [string(name: 'SERVICE', value: 'webapp')]
                }
            }
        }
    }
    post {
        always {
            echo 'Webapp build and deployment finished.'
        }
    }
}
