pipeline {
    agent any  // This specifies that Jenkins can run the job on any available agent

    environment {
        // Setting environment variables for Docker, SonarQube, and your servers
        DOCKER_IMAGE = "my-app"  // Name of the Docker image to build and deploy
        STAGING_SERVER = "staging-server-address"  // Replace with actual staging server address
        PRODUCTION_SERVER = "production-server-address"  // Replace with actual production server address
        SONAR_HOST_URL = "http://your-sonarqube-server"  // URL of your SonarQube server
        SONAR_PROJECT_KEY = "your-project-key"  // Replace with your SonarQube project key
        SONAR_TOKEN = credentials('sonarqube-token')  // Use Jenkins credentials for SonarQube token
    }

    stages {

        // Step 1: Build Stage
        stage('Build') {
            steps {
                echo 'Building the application...'
                // This example assumes a Node.js project. Modify it to fit your project (e.g., Maven, Gradle)
                sh 'npm install'  // Install dependencies
                sh 'npm run build'  // Build the project (e.g., run your build command)
            }
        }

        // Step 2: Test Stage
        stage('Test') {
            steps {
                echo 'Running automated tests...'
                // Running tests. Change the command according to your testing framework (e.g., JUnit, Selenium, etc.)
                sh 'npm test'  // For a Node.js project, modify this for your test framework
            }
        }

        // Step 3: Code Quality Analysis (SonarQube)
        stage('Code Quality Analysis') {
            steps {
                echo 'Running code quality analysis using SonarQube...'
                // Running SonarQube scanner to analyze the code
                sh """
                sonar-scanner \
                  -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                  -Dsonar.sources=. \
                  -Dsonar.host.url=${SONAR_HOST_URL} \
                  -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        // Step 4: Deploy to Staging
        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to staging environment...'
                // Example Docker deployment to a staging environment
                sh """
                docker build -t ${DOCKER_IMAGE} .  // Build the Docker image
                docker run -d -p 80:80 --name ${DOCKER_IMAGE}_staging ${DOCKER_IMAGE}  // Run it on port 80
                """
            }
        }

        // Step 5: Release to Production
        stage('Release to Production') {
            steps {
                echo 'Releasing to production environment...'
                // SSH into the production server and deploy the application
                sh """
                ssh user@${PRODUCTION_SERVER} 'docker pull ${DOCKER_IMAGE} && docker stop ${DOCKER_IMAGE}_prod && docker rm ${DOCKER_IMAGE}_prod && docker run -d -p 80:80 --name ${DOCKER_IMAGE}_prod ${DOCKER_IMAGE}'
                """
            }
        }

        // Step 6: Monitoring and Alerts
        stage('Monitoring and Alerts') {
            steps {
                echo 'Setting up monitoring for production...'
                // Set up monitoring or alerts using a tool like Datadog or New Relic.
                // You could run monitoring setup commands or send API calls to monitoring services here.
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()  // Clean up the workspace after the job completes
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs for more details.'
        }
    }
}
