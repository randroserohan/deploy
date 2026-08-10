pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out website source code...'
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                echo 'Validating website...'

                sh '''
                    test -f index.html
                    test -f about.html
                    test -f blog.html
                    test -f contact.html
                    test -f product.html
                    test -f singlepost.html

                    echo "Website validation successful"
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying complete website...'

                sh '''
                    sudo /usr/local/bin/deploy-website
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
            echo '========================================'
            echo '   WEBSITE DEPLOYMENT SUCCESSFUL'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo '   DEPLOYMENT FAILED'
            echo '========================================'
        }
    }
}
