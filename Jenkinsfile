pipeline {

    agent any

    environment {
        IMAGE_NAME = 'frozen-yogurt-website'
        CONTAINER_NAME = 'frozen-yogurt'
        BACKUP_NAME = 'frozen-yogurt-backup'
        HOST_PORT = '8081'
    }

    options {
        timestamps()
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                echo '========================================'
                echo 'CHECKING OUT WEBSITE FROM GITHUB'
                echo '========================================'

                checkout scm
            }
        }

        stage('Validate') {
            steps {
                echo '========================================'
                echo 'VALIDATING WEBSITE'
                echo '========================================'

                sh '''
                    test -f index.html
                    test -f about.html
                    test -f blog.html
                    test -f contact.html
                    test -f product.html
                    test -f singlepost.html

                    test -d css
                    test -d js
                    test -d images
                    test -d fonts

                    echo "Website validation successful"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '========================================'
                echo 'BUILDING DOCKER IMAGE'
                echo '========================================'

                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                        -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Security Scan') {
            steps {
                echo '========================================'
                echo 'TRIVY SECURITY SCAN'
                echo '========================================'

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
                echo '========================================'
                echo 'BACKING UP CURRENT CONTAINER'
                echo '========================================'

                script {
                    sh '''
                        # Remove old backup if it exists
                        if docker ps -a --format '{{.Names}}' | grep -q "^${BACKUP_NAME}$"; then
                            docker rm -f ${BACKUP_NAME}
                        fi

                        # If current container exists
                        if docker ps -a --format '{{.Names}}' | grep -q "^${CONTAINER_NAME}$"; then

                            # Stop current container
                            docker stop ${CONTAINER_NAME} || true

                            # Rename it as backup
                            docker rename ${CONTAINER_NAME} ${BACKUP_NAME}

                            echo "Previous container backed up"

                        else
                            echo "No previous container found"
                        fi
                    '''

                    env.BACKUP_CREATED = 'true'
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '========================================'
                echo 'DEPLOYING NEW WEBSITE'
                echo '========================================'

                sh '''
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

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
                echo '========================================'
                echo 'CHECKING WEBSITE HEALTH'
                echo '========================================'

                sh '''
                    echo "Waiting for Nginx..."
                    sleep 5

                    curl -f http://localhost:${HOST_PORT}/ > /dev/null

                    echo "========================================"
                    echo "HEALTH CHECK PASSED"
                    echo "WEBSITE IS UP"
                    echo "========================================"
                '''
            }
        }

        stage('Remove Old Backup') {
            steps {
                echo '========================================'
                echo 'REMOVING OLD BACKUP'
                echo '========================================'

                sh '''
                    if docker ps -a --format '{{.Names}}' | grep -q "^${BACKUP_NAME}$"; then
                        docker rm -f ${BACKUP_NAME}
                        echo "Old backup removed"
                    else
                        echo "No old backup found"
                    fi
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'DEPLOYMENT SUCCESSFUL'
            echo '========================================'
            echo "Website: http://localhost:${HOST_PORT}"
            echo "Docker Image: ${IMAGE_NAME}:${BUILD_NUMBER}"
            echo 'Health Check: PASSED'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'DEPLOYMENT FAILED'
            echo 'STARTING AUTOMATIC ROLLBACK'
            echo '========================================'

            sh '''
                # Remove failed new container
                if docker ps -a --format '{{.Names}}' | grep -q "^${CONTAINER_NAME}$"; then
                    docker rm -f ${CONTAINER_NAME}
                fi

                # Restore previous container
                if docker ps -a --format '{{.Names}}' | grep -q "^${BACKUP_NAME}$"; then

                    docker rename ${BACKUP_NAME} ${CONTAINER_NAME}

                    docker start ${CONTAINER_NAME}

                    echo "========================================"
                    echo "ROLLBACK SUCCESSFUL"
                    echo "PREVIOUS WEBSITE RESTORED"
                    echo "========================================"

                else
                    echo "No backup container available"
                    echo "Nothing to rollback"
                fi
            '''
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
