pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Checking project files..."
                    ls -la
                    test -f hello.html
                    echo "HTML file found successfully."
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying website..."
                    cp hello.html /var/www/html/index.html
                    echo "Deployment completed."
                '''
            }
        }
    }
}
