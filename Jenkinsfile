pipeline {
    options { disableConcurrentBuilds() }

    agent {
        // Custom image that has BOTH docker cli AND git
        docker {
            image 'docker:24.0-cli-git'
            args  '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        IMAGE_NAME     = 'my-flask-app'
        IMAGE_TAG      = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = "flask-app-${env.BUILD_NUMBER}"
        GIT_REPO       = 'https://github.com/kingofshades/jenkins_test.git'
        GIT_CREDS      = 'github-pat'        // Jenkins credentials ID
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/']],
                          userRemoteConfigs: [[url: env.GIT_REPO,
                                               credentialsId: env.GIT_CREDS]]])
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                // optional "latest" tag
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy') {
            steps {
                sh """
                   docker rm -f flask-app || true  
                   docker run -d --name flask-app \
                       -p 5000:5000 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success { echo "🌐 http://localhost:5000/ is now serving build #${IMAGE_TAG}" }
        cleanup { sh 'docker image prune -f || true' }   // reclaim space
    }
}
