pipeline {
    agent any

    environment {
        APP_NAME = 'onlinebookstore'
        DOCKER_IMAGE = "your-dockerhub-username/${APP_NAME}"
        BRANCH_NAME = "${env.GIT_BRANCH}".replaceAll('/', '-')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests=false'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }

            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BRANCH_NAME}")
                }
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${BRANCH_NAME}"
                }
            }
        }

        stage('Deploy to Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['your-ssh-credential-id']) {
                    sh """
                    ssh user@your-server-ip 'docker pull ${DOCKER_IMAGE}:${BRANCH_NAME} && docker stop ${APP_NAME} || true && docker rm ${APP_NAME} || true && docker run -d --name ${APP_NAME} -p 8080:8080 ${DOCKER_IMAGE}:${BRANCH_NAME}'
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build, test, and deployment successful!'
        }
        failure {
            echo '❌ Build or test failed.'
        }
    }
}
