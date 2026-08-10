pipeline {

    agent any

    environment {
        IMAGE_NAME     = "frozen-yogurt-website"
        CONTAINER_NAME = "frozen-yogurt"
        BACKUP_NAME    = "frozen-yogurt-backup"
        HOST_PORT      = "8081"
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

        stage('Security Scan') {
            steps {
                echo 'Scanning Docker image with Trivy...'

                sh '''
                    trivy image \
                       --severity HIGH,CRITICAL \
                       --exit-code 1 \
                       ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Backup Current Container') {
            steps {
                echo 'Backing up current container...'

                sh '''
                    # Remove old backup if it exists
                    docker rm -f ${BACKUP_NAME} 2>/dev/null || true

                    # If current container exists
                    if docker ps -a --format '{{.Names}}' | grep -q "^${CONTAINER_NAME}$"; then

                        # Stop old container first
                        docker stop ${CONTAINER_NAME} 2>/dev/null || true

                        # Rename it as backup
                        docker rename \
                            ${CONTAINER_NAME} \
                            ${BACKUP_NAME}

                        echo "Previous container stopped and backed up"

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
                    echo "Waiting for website..."
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
                    if docker ps -a --format '{{.Names}}' | grep -q "^${BACKUP_NAME}$"; then

                        docker rm -f ${BACKUP_NAME}

                        echo "Old container backup removed"

                    else

                        echo "No backup container to remove"

                    fi
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo '       DEPLOYMENT SUCCESSFUL'
            echo '========================================'

            sh '''
                echo "Running container:"
                docker ps --format "table {{.Names}}\\t{{.Image}}\\t{{.Ports}}\\t{{.Status}}"
            '''
        }

        failure {
            echo '========================================'
            echo '       DEPLOYMENT FAILED'
            echo '       STARTING ROLLBACK'
            echo '========================================'

            sh '''
                # Remove failed new container
                docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                # Restore previous container
                if docker ps -a --format '{{.Names}}' | grep -q "^${BACKUP_NAME}$"; then

                    echo "Restoring previous website..."

                    docker rename \
                        ${BACKUP_NAME} \
                        ${CONTAINER_NAME}

                    docker start ${CONTAINER_NAME}

                    echo "Previous website restored successfully"

                else

                    echo "No backup container available for rollback"

                fi
            '''
        }
    }
}
