pipeline {

    agent any

    environment {
        IMAGE_NAME = "frozen-yogurt-website"
        CONTAINER_NAME = "frozen-yogurt"
        HOST_PORT = "8081"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out website from GitHub...'
            }
        }

        stage('Validate') {
            steps {
                echo 'Validating website files...'

                sh '''
                    test -f index.html
                    test -d css
                    test -d js
                    test -d images

                    echo "Website validation successful"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                    -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Backup Current Container') {
            steps {
                echo 'Backing up current container...'

                sh '''
                    if docker ps -a --format '{{.Names}}' | grep -q "^${CONTAINER_NAME}$"; then

                        docker rm -f ${CONTAINER_NAME}-backup 2>/dev/null || true

                        docker rename \
                        ${CONTAINER_NAME} \
                        ${CONTAINER_NAME}-backup

                        echo "Previous container backed up"

                    else

                        echo "No previous container found"

                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying new website container...'

                sh '''
                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p ${HOST_PORT}:80 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}

                    echo "New website container started"
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking website health...'

                sh '''
                    sleep 5

                    curl -f http://localhost:${HOST_PORT} > /dev/null

                    echo "Website health check PASSED"
                '''
            }
        }

        stage('Cleanup Backup') {
            steps {
                echo 'Removing old container backup...'

                sh '''
                    docker rm -f ${CONTAINER_NAME}-backup 2>/dev/null || true
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'DEPLOYMENT SUCCESSFUL'
            echo '========================================'

            sh '''
                docker ps
            '''
        }

        failure {
            echo '========================================'
            echo 'DEPLOYMENT FAILED - ROLLBACK'
            echo '========================================'

            sh '''
                docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                if docker ps -a --format '{{.Names}}' | grep -q "^${CONTAINER_NAME}-backup$"; then

                    docker rename \
                    ${CONTAINER_NAME}-backup \
                    ${CONTAINER_NAME}

                    docker start ${CONTAINER_NAME}

                    echo "Previous website restored"

                else

                    echo "No backup container available"

                fi
            '''
        }
    }
}
