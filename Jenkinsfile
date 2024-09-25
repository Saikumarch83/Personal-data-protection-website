pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'personal-data-protection-app:latest'  // Docker image name for your app
        GIT_URL = 'https://github.com/Saikumarch83/Personal-data-protection-website.git'  // Your GitHub repository
        SONARQUBE_HOST_URL = 'http://your-sonarqube-server'  // SonarQube server URL
        SONARQUBE_PROJECT_KEY = 'personal-data-protection-project'  // SonarQube project key
        SONARQUBE_TOKEN = credentials('sonarqube-token')  // SonarQube token (set up in Jenkins credentials)
    }

    stages {
        
        // Clone the repository
        stage('Checkout') {
            steps {
                echo 'Checking out the project...'
                git branch: 'main', url: "${GIT_URL}"
            }
        }

        // Build the project
        stage('Build') {
            steps {
                echo 'Building the project...'
                // Assuming Node.js project with npm
                sh 'npm install'  // Install dependencies
                sh 'npm run build'  // Build the project
            }
        }

        // Test the project
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'  // Run the project's test cases
            }
        }

        // Code Quality Analysis using SonarQube
        stage('Code Quality Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {  // Ensure SonarQube is configured in Jenkins
                    sh """
                    sonar-scanner \
                        -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONARQUBE_HOST_URL} \
                        -Dsonar.login=${SONARQUBE_TOKEN}
                    """
                }
            }
        }

        // Deploy to Staging
        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to staging environment...'
                // Assuming Docker is used for deployment
                sh """
                docker build -t ${DOCKER_IMAGE} .
                docker run -d -p 8081:80 --name personal-data-protection-staging ${DOCKER_IMAGE}
                """
            }
        }

        // Release to Production
        stage('Release to Production') {
            steps {
                echo 'Releasing to production...'
                // Stop and remove the previous production container, then start the new one
                sh """
                docker stop personal-data-protection-prod || true
                docker rm personal-data-protection-prod || true
                docker run -d -p 8080:80 --name personal-data-protection-prod ${DOCKER_IMAGE}
                """
            }
        }

        // Serve on localhost:8080
        stage('Serve HTML on Localhost') {
            steps {
                echo 'Serving the HTML file on localhost:8080...'
                sh """
                # Assuming your Docker container is already serving the application on port 80
                echo 'Your application is now accessible at http://localhost:8080'
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()  // Clean up the Jenkins workspace after the build
        }
        success {
            echo 'Pipeline completed successfully!'
            echo 'Visit your application at http://localhost:8080'
        }
        failure {
            echo 'Pipeline failed! Check the logs for more details.'
        }
    }
}
