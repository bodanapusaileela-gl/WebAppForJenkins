pipeline {
    agent any

    tools {
        maven 'mymaveen'
    }

    environment {
        APP_SERVER_IP = "34.217.45.8"
        APP_NAME      = "myapp"
        IMAGE_NAME    = "bodanapusaileela/myapp"
        IMAGE_TAG     = "latest"
        DOCKER_CREDS  = "DOCKER_CREDS"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/bodanapusaileela-gl/WebAppForJenkins.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Verify Artifact') {
            steps {
                sh 'ls -lh target'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                  docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Docker Login & Push') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'DOCKER_CREDS',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
              echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
              docker push bodanapusaileela/myapp:latest
            '''
        }
    }
}
        stage('Deploy to Application Server') {
    steps {
        sshagent(['ec2-ssh-key']) {
            withCredentials([usernamePassword(
                credentialsId: "DOCKER_CREDS",
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                sh """
                ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER_IP} '
                  echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin &&
                  docker pull ${IMAGE_NAME}:${IMAGE_TAG} &&
                  docker stop ${APP_NAME} || true &&
                  docker rm ${APP_NAME} || true &&
                  docker run -d --name ${APP_NAME} -p 8081:8080 ${IMAGE_NAME}:${IMAGE_TAG}
                '
                """
            }
        }
    }
 }
} }
