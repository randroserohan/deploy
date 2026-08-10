pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                echo 'Validating HTML...'

                sh '''
                    test -f hello.html
                    echo "HTML validation successful"
                '''
            }
        }

        stage('Backup') {
            steps {
                echo 'Backing up current website...'

                sh '''
                    if [ -f /var/www/html/index.html ]; then
                        sudo cp /var/www/html/index.html /var/www/html/index.html.backup
                        echo "Backup created"
                    else
                        echo "No existing website to backup"
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying new version...'

                sh '''
                    sudo cp hello.html /var/www/html/index.html
                    sudo chmod 644 /var/www/html/index.html
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking website health...'

                sh '''
                    curl -f http://localhost > /dev/null
                    echo "Website health check PASSED"
                '''
            }
        }
    }

    post {

        success {
            echo '================================='
            echo 'DEPLOYMENT SUCCESSFUL'
            echo '================================='
        }

        failure {
            echo '================================='
            echo 'DEPLOYMENT FAILED - ROLLBACK'
            echo '================================='

            sh '''
                if [ -f /var/www/html/index.html.backup ]; then
                    sudo cp /var/www/html/index.html.backup /var/www/html/index.html
                    echo "Previous version restored"
                fi
            '''
        }
    }
}
