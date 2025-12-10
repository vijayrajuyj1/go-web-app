pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "vijayarajult2/go-web-app:${BUILD_NUMBER}"
        GIT_REPO_NAME = "go-web-app"
        GIT_USER_NAME = "vijayrajuyj1"
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout your Git repo
               //  git branch: 'main', url: 'https://github.com/vijayarajuyj1/go-web-app.git'
                   echo "repo already checked"
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build Docker image from your Dockerfile
                    def dockerImage = docker.build("${DOCKER_IMAGE}")
                    // Push Docker image to Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Update values.yaml and Push') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "vijayarajuyj1@gmail.com"
                        git config user.name "vijayarajuyj1"
                        sed -i 's/tag: .*/tag: '"${BUILD_NUMBER}"'/g' helm/helm1/values.yaml
                        git add helm/helm1/values.yaml
                        git commit -m "Update image tag to ${BUILD_NUMBER}" || echo "No changes to commit"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
}
