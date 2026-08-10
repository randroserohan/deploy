pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out website...'
                checkout scm
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

        stage('Backup') {
            steps {
                echo 'Backing up current website...'

                sh '''
                    if [ -d /var/www/html ]; then
                        sudo cp -a /var/www/html /var/www/html.backup
                        echo "Website backup created"
                    fi
                '''
            }
        }

        stage('Deploy Website') {
            steps {
                echo 'Deploying complete website...'

                sh '''
                    sudo rm -rf /var/www/html/*
                    sudo cp -a . /var/www/html/

                    sudo rm -rf /var/www/html/.git

                    sudo chown -R www-data:www-data /var/www/html
                    sudo find /var/www/html -type d -exec chmod 755 {} \\;
                    sudo find /var/www/html -type f -exec chmod 644 {} \\;

                    echo "Complete website deployed successfully"
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking website...'

                sh '''
                    curl -f http://localhost > /dev/null
                    echo "Website health check PASSED"
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo ' COMPLETE WEBSITE DEPLOYMENT SUCCESSFUL'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo ' DEPLOYMENT FAILED - RESTORING BACKUP'
            echo '========================================'

            sh '''
                if [ -d /var/www/html.backup ]; then
                    sudo rm -rf /var/www/html
                    sudo cp -a /var/www/html.backup /var/www/html
                    echo "Previous website restored"
                fi
            '''
        }
    }
}
