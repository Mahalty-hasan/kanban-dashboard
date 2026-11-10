pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mahaltyhasan/kanban-dashboard'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        CONTAINER_NAME = 'kanban-app'
        APP_PORT = '8080'
        PREVIOUS_IMAGE_FILE = "/tmp/kanban-previous-image-${BUILD_NUMBER}"
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
                    env.IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                    sh 'docker build -t $DOCKER_IMAGE:$IMAGE_TAG .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push "$DOCKER_IMAGE:$IMAGE_TAG"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    docker inspect --format='{{.Config.Image}}' "$CONTAINER_NAME" \
                        > "$PREVIOUS_IMAGE_FILE" 2>/dev/null || true

                    docker pull "$DOCKER_IMAGE:$IMAGE_TAG"

                    docker rm -f "$CONTAINER_NAME" 2>/dev/null || true

                    docker run -d \
                        --name "$CONTAINER_NAME" \
                        --restart unless-stopped \
                        -p 80:$APP_PORT \
                        "$DOCKER_IMAGE:$IMAGE_TAG"
                '''
            }
        }

	stage('Rollback Test'){
	    steps{
		sh 'echo "Testing rollback..." && exit 1'
	    }
	}

        stage('Health Check') {
            steps {
                sh '''
                    sleep 10
                    docker inspect --format='{{.State.Health.Status}}' "$CONTAINER_NAME" | grep -q healthy
                    curl --fail --silent http://localhost/ > /dev/null
                '''
            }
        }
    }

    post {
        failure {
            script {
                sh '''
                    echo "Deployment failed. Starting rollback..."

                    if [ -s "$PREVIOUS_IMAGE_FILE" ]; then
                        PREVIOUS_IMAGE=$(cat "$PREVIOUS_IMAGE_FILE")

                        docker rm -f "$CONTAINER_NAME" 2>/dev/null || true

                        docker run -d \
                            --name "$CONTAINER_NAME" \
                            --restart unless-stopped \
                            -p 80:$APP_PORT \
                            "$PREVIOUS_IMAGE"

                        sleep 10

                        docker inspect --format='{{.State.Health.Status}}' "$CONTAINER_NAME"
                        echo "Rollback attempted."
                    else
                        echo "No previous image found. Nothing to roll back to."
                    fi
                '''
            }
        }

        success {
            echo 'Kanban application deployed successfully!'
        }
    }
}
