pipeline {
    agent none

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "guntivinithchandra/devops-project1"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            agent any
            steps {
                git branch: 'main', url: 'https://github.com/vinithchandra/-CI-CD-Pipeline-with-Jenkins-Docker-Kubernetes.git'
            }
        }

        stage('Run Tests') {
            agent {
                docker {
                    image 'node:18'
                }
            }
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            agent any
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push to DockerHub') {
            agent any
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Deploy to Kubernetes') {
            agent any
            steps {
                sh "sed -i 's|IMAGE_PLACEHOLDER|$IMAGE_NAME:$IMAGE_TAG|' k8s/deployment.yaml"
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! App is live.'
        }
        failure {
            echo 'Pipeline failed. Check logs above.'
        }
    }
}
