pipeline {
    agent any
    
    environment {
        // Docker and SonarQube settings
        DOCKER_IMAGE = "my-app"
        STAGING_SERVER = "staging-server-address"  // Replace with your actual staging server address
        PRODUCTION_SERVER = "production-server-address"  // Replace with your production server address
        SONAR_HOST_URL = "http://your-sonarqube-server"  // SonarQube URL
        SONAR_PROJECT_KEY = "your-project-key"  // Your SonarQube project key
        SONAR_TOKEN = credentials('sonarqube-token')  // Jenkins credentials for SonarQube token
    }

    stages {
        
        // Build Stage: Compile and create the artifact (Docker image, JAR file, etc.)
        stage('Build') {
            steps {
                echo 'Building the application...'
                // Example for a Node.js project (change to your build system, e.g., Maven for Java projects)
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        // Test Stage: Run automated tests to validate the application
        stage('Test') {
            steps {
                echo 'Running automated tests...'
                // Example for running Node.js tests (modify this for your framework, such as JUnit, Selenium, etc.)
                sh 'npm test'
            }
        }

        // Code Quality Analysis Stage: Run SonarQube to check code quality
        stage('Code Quality Analysis') {
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
        
        // Deploy to Staging Stage: Deploy the application to a test/staging environment
        stage('Deploy to Staging') {
            steps {
                echo 'Deploying application to Staging environment...'
                // Example of deploying a Docker image to a staging environment
                sh """
                docker build -t ${DOCKER_IMAGE} .
                docker run -d -p 80:80 --name ${DOCKER_IMAGE}_staging ${DOCKER_IMAGE}
                """
            }
        }

        // Release Stage: Promote the application to production environment
        stage('Release to Production') {
            steps {
                echo 'Releasing application to Production...'
                // Example: SSH into the production server and deploy the application
                sh """
                ssh user@${PRODUCTION_SERVER} 'docker pull ${DOCKER_IMAGE} && docker stop ${DOCKER_IMAGE}_prod && docker rm ${DOCKER_IMAGE}_prod && docker run -d -p 80:80 --name ${DOCKER_IMAGE}_prod ${DOCKER_IMAGE}'
                """
            }
        }

        // Monitoring and Alerting Stage: Set up monitoring and alerting for production
        stage('Monitoring and Alerts') {
            steps {
                echo 'Setting up monitoring for production...'
                // Example for monitoring integration (Datadog, New Relic, Prometheus, etc.)
                // You can include commands or API integrations for monitoring here
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()  // Cleans up the workspace after every build
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed! Please check the logs for more details.'
        }
    }
}
