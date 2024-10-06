pipeline {
    agent {
        label 'node1-build'  // Use the specified agent with the label 'node1-build'
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the code from Git
                    sh 'echo Checkout started'
                    // Uncomment the line below to enable checkout
                    // git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
                }
            }
        }
        stage('Install Go') {
            steps {
                script {
                    // Install Go if it's not already installed
                    sh 'sudo apt update'
                    sh 'sudo apt install -y golang-go'
                }
            }
        }
        stage('Build and Test') {
            steps {
                script {
                    // List files for debugging purposes
                    sh 'ls -ltr'
                    // Set Go modules path to ensure they are installed correctly
                    sh 'go mod tidy'  // Ensure Go modules are tidy
                    // Download Go modules and build the project
                    sh 'go build -o main ./...' // Build the Go application
                    // Run tests if applicable
                    sh 'go test ./...' // Run Go tests
                }
            }
        }
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://44.222.203.42:9000"  // Define SonarQube URL
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                        // Run SonarQube analysis
                        sh '''
                            sonar-scanner \
                            -Dsonar.projectKey=go-web-app \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dsonar.login=$SONAR_AUTH_TOKEN
                        '''
                    }
                }
            }
        }
        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "vijayarajult2/go-web-app:${BUILD_NUMBER}"  // Define the Docker image name
                REGISTRY_CREDENTIALS = credentials('docker-cred')  // Docker credentials
            }
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    // Push the Docker image to the registry
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "go-web-app"  // Define the Git repository name
                GIT_USER_NAME = "vijayrajuyj1"  // Define the Git user name
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        // Configure Git user details and update the deployment file
                        sh '''
                            git config --global user.email "vijayarajuyj1@gmail.com"
                            git config --global user.name "${GIT_USER_NAME}"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            sed -i "s/{{ .Values.image.tag }}/${BUILD_NUMBER}/g" k8s/deployment.yml
                            git add k8s/deployment.yml
                            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
