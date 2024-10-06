pipeline {
  agent {
    label 'node1-build'
  }

  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        // Uncomment and use the line below to checkout the actual repository
        // git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
      }
    }

    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // Install Go
        sh 'sudo apt install -y golang-go'
        // Build the Go project
        sh 'go build -o main'
      }
    }

    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "vijayarajult2/go-web-app:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            // Build Docker image
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            // Push Docker image to Docker Hub
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }

    stage('Update Values.yaml File') {
      environment {
        GIT_REPO_NAME = "go-web-app"
        GIT_USER_NAME = "vijayrajuyj1"
      }
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          sh '''
              git config user.email "vijayarajuyj1@gmail.com"
              git config user.name "vijayrajuyj1"
              BUILD_NUMBER=${BUILD_NUMBER}
              # Update the image tag in values.yaml
              sed -i 's/tag: .*/tag: '"${BUILD_NUMBER}"'/' helm/helm1/values.yaml
              # Check git status to see if the file was modified
              git status
              git add helm/helm1/values.yaml
              git commit -m "Update image tag in values.yaml to version ${BUILD_NUMBER}" || echo "No changes to commit"
              git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          '''
        }
      }
    }
  }
}
