pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-app"
        DOCKERHUB_REPO = "saichowdary009/python-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "flask-app"
    }

    stages {

        stage('Build') {
            steps {
                sh 'python3 -m py_compile app.py'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        // 🔥 This replaces your old Docker Run (temporary test)
        stage('Deploy') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                docker run -d -p 5000:5000 --name $CONTAINER_NAME $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Docker Tag & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKERHUB_REPO:$IMAGE_TAG
                    docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKERHUB_REPO:latest

                    docker push $DOCKERHUB_REPO:$IMAGE_TAG
                    docker push $DOCKERHUB_REPO:latest

                    docker logout
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo '✅ App deployed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
