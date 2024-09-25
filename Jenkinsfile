pipeline {
    agent any
    
    environment {
        // Define environment variables (if necessary)
        DOCKER_IMAGE = "my-app"
        STAGING_SERVER = "staging-server-address" // Replace with your staging server address
        PRODUCTION_SERVER = "production-server-address" // Replace with your production server address
        SONAR_HOST_URL = "http://your-sonarqube-server"
        SONAR_PROJECT_KEY = "personal-data-protection-website"
        SONAR_TOKEN = credentials('sonarqube-token') // Ensure you have SonarQube credentials set in Jenkins
    }

    stages {
        
        // Build stage: Install dependencies and build the project
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        // Test stage: Run unit tests
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }
        
        // Code Quality stage: Analyze code quality with SonarQube
        stage('Code Quality') {
            steps {
                echo 'Running code quality analysis with SonarQube...'
                sh """
                sonar-scanner \
                  -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                  -Dsonar.sources=. \
                  -Dsonar.host.url=${SONAR_HOST_URL} \
                  -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }
        
        // Deploy stage: Deploy to staging environment
        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to staging environment...'
                sh """
                docker build -t ${DOCKER_IMAGE} .
                docker run -d -p 80:80 --name ${DOCKER_IMAGE}_staging ${DOCKER_IMAGE}
                """
            }
        }
        
        // Release stage: Promote and deploy to production
        stage('Release to Production') {
            steps {
                echo 'Releasing to production...'
                sh """
                ssh user@${PRODUCTION_SERVER} 'docker pull ${DOCKER_IMAGE} && docker stop ${DOCKER_IMAGE}_prod && docker rm ${DOCKER_IMAGE}_prod && docker run -d -p 80:80 --name ${DOCKER_IMAGE}_prod ${DOCKER_IMAGE}'
                """
            }
        }

        // Monitoring stage: Set up monitoring (example: with a tool like Datadog)
        stage('Monitoring') {
            steps {
                echo 'Setting up monitoring...'
                // Example: send a monitoring notification or set up an alert
                // You can integrate with external tools like Datadog or New Relic here
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            // Clean workspace after build
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed! Please check logs for details.'
        }
    }
}
