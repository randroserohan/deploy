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
                echo 'Validating HTML files...'

                sh '''
                    test -f hello.html
                    echo "hello.html found successfully"
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying website to Nginx...'

                sh '''
                    sudo cp hello.html /var/www/html/index.html
                    sudo chmod 644 /var/www/html/index.html
                '''
            }
        }

        stage('Verify') {
            steps {
                echo 'Verifying deployment...'

                sh '''
                    curl -f http://localhost/ > /dev/null
                    echo "Website is responding successfully"
                '''
            }
        }
    }

    post {
        success {
            echo '🎉 Deployment successful!'
        }

        failure {
            echo '❌ Deployment failed!'
        }
    }
}
