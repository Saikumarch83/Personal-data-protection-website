pipeline {
    agent any
    
    environment {
        // Environment variables for Docker and SonarQube
        DOCKER_IMAGE = "my-app"
        STAGING_SERVER = "staging-server-address"  // Replace with your staging server address
        PRODUCTION_SERVER = "production-server-address"  // Replace with your production server address
        SONAR_HOST_URL = "http://your-sonarqube-server"  // SonarQube server URL
        SONAR_PROJECT_KEY = "personal-data-protection-website"
        SONAR_TOKEN = credentials('sonarqube-token')  // SonarQube credentials (configure this in Jenkins)
    }

    stages {
        
        // Build Stage: Install dependencies and build the project
        stage('Build') {
            steps {
                echo 'Building the application...'
                // Example for Node.js project; modify according to your project's needs
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        // Test Stage: Run automated tests
        stage('Test') {
            steps {
                echo 'Running tests...'
                // Example: Run unit tests with a Node.js testing framework (like Jest or Mocha)
                sh 'npm test'
            }
        }
        
        // Code Quality Analysis Stage: Analyze code with SonarQube
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
        
        // Deploy Stage: Deploy to a staging environment using Docker
        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to Staging environment...'
                // Building a Docker image and running it as a container in the staging environment
                sh """
                docker build -t ${DOCKER_IMAGE} .
                docker run -d -p 80:80 --name ${DOCKER_IMAGE}_staging ${DOCKER_IMAGE}
                """
            }
        }
        
        // Release Stage: Deploy to production
        stage('Release to Production') {
            steps {
                echo 'Releasing to Production environment...'
                // Example: Using SSH to deploy to a production server
                sh """
                ssh user@${PRODUCTION_SERVER} 'docker pull ${DOCKER_IMAGE} && docker stop ${DOCKER_IMAGE}_prod && docker rm ${DOCKER_IMAGE}_prod && docker run -d -p 80:80 --name ${DOCKER_IMAGE}_prod ${DOCKER_IMAGE}'
                """
            }
        }

        // Monitoring Stage: Monitor the application in production (optional, but recommended for HD grade)
        stage('Monitoring and Alerts') {
            steps {
                echo 'Setting up monitoring...'
                // Example: You can integrate Datadog, New Relic, or any other monitoring tool
                // This step can be customized to add alert mechanisms like emails, Slack, or PagerDuty
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            node {
                cleanWs()  // Clean up workspace after every build
            }
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed! Please check logs for details.'
        }
    }
}
