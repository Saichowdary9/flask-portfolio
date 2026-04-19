pipeline {
    agent { label 'worker-node' }

    environment {
        IMAGE_NAME = "python-app"
        DOCKERHUB_REPO = "Saichowdary9/python-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Github Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/theshubhamgour/jenkins-tutorial.git'
                sh 'pwd'
            }
        }

        stage('Build') {
            steps {
                dir('4-python-jenkins-docker-app') {
                    sh 'python3 -m py_compile app.py'
                }
            }
        }

        stage('Test') {
            steps {
                dir('4-python-jenkins-docker-app') {
                    sh 'python3 -m unittest discover tests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('4-python-jenkins-docker-app') {
                    sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }

        stage('Docker Run') {
            steps {
                dir('4-python-jenkins-docker-app') {
                    sh 'docker run --rm $IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }

        stage('Docker Tag & Push') {
            steps {
                dir('4-python-jenkins-docker-app') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        
                        # Tag with build number
                        docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKERHUB_REPO:$IMAGE_TAG
                        
                        # Tag as latest
                        docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKERHUB_REPO:latest
                        
                        # Push both tags
                        docker push $DOCKERHUB_REPO:$IMAGE_TAG
                        docker push $DOCKERHUB_REPO:latest
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
