pipeline {
  agent {
    label 'node1-build'
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'http://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'sudo apt install golang-go'
        sh 'go build -o main'
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "vijayarajult2/go-web-app:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-ap/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
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
                    sed -i 's/tag: .*/tag: "${BUILD_NUMBER}"/' helm/helm1/values.yaml
                    git add helm/helm1/values.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
