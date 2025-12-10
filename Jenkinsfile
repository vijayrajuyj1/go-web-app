pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "vijayarajult2/go-web-app:${BUILD_NUMBER}"
    }

    triggers {
        githubPush() // Trigger on GitHub push
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }
    }
}
