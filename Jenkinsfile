pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "my-app"
        STAGING_SERVER = "staging-server-address"
        PRODUCTION_SERVER = "production-server-address"
        SONAR_HOST_URL = "http://your-sonarqube-server"
        SONAR_PROJECT_KEY = "your-project-key"
        SONAR_TOKEN = credentials('sonarqube-token')
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                echo 'Running automated tests...'
                sh 'npm test'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                sh """
                sonar-scanner \
                  -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                  -Dsonar.sources=. \
                  -Dsonar.host.url=${SONAR_HOST_URL} \
                  -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to Staging environment...'
                sh """
                docker build -t ${DOCKER_IMAGE} .
                docker run -d -p 80:80 --name ${DOCKER_IMAGE}_staging ${DOCKER_IMAGE}
                """
            }
        }

        stage('Release to Production') {
            steps {
                echo 'Releasing to Production environment...'
                sh """
                ssh user@${PRODUCTION_SERVER} 'docker pull ${DOCKER_IMAGE} && docker stop ${DOCKER_IMAGE}_prod && docker rm ${DOCKER_IMAGE}_prod && docker run -d -p 80:80 --name ${DOCKER_IMAGE}_prod ${DOCKER_IMAGE}'
                """
            }
        }

        stage('Serve HTML on Localhost') {
            steps {
                echo 'Serving HTML file on localhost:8080...'
                // Start a Python HTTP server to serve the build directory
                sh """
                cd build  # Navigate to the directory containing the HTML files (modify as necessary)
                nohup python3 -m http.server 8080 &  # Run HTTP server in the background
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
            // You can also add a step to display the URL to access the HTML
            echo 'You can view the HTML report at http://localhost:8080'
        }
        failure {
            echo 'Pipeline failed! Please check logs for details.'
        }
    }
}
