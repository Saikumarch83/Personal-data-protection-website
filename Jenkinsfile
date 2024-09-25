pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'personal-data-protection-app:latest'  // Docker image name for your app
        GIT_URL = 'https://github.com/Saikumarch83/Personal-data-protection-website.git'  // Your GitHub repository
        SONARQUBE_HOST_URL = 'http://your-sonarqube-server'  // SonarQube server URL
        SONARQUBE_PROJECT_KEY = 'personal-data-protection-project'  // SonarQube project key
        SONARQUBE_TOKEN = credentials('sonarqube-token')  // SonarQube token (ensure it's set up in Jenkins)
    }

    stages {

        // Checkout code from the GitHub repository
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
                sh 'npm install'  // Install dependencies for a Node.js project
                sh 'npm run build'  // Build the project
            }
        }

        // Run tests
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'  // Run the tests
            }
        }

        // Run SonarQube for code quality analysis
        stage('Code Quality Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {  // Ensure that 'SonarQube' is configured in Jenkins
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

        // Deploy to staging environment
        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to staging environment...'
                sh """
                docker build -t ${DOCKER_IMAGE} .
                docker run -d -p 8081:80 --name personal-data-protection-staging ${DOCKER_IMAGE}
                """
            }
        }

        // Release to production environment
        stage('Release to Production') {
            steps {
                echo 'Releasing to production...'
                sh """
                docker stop personal-data-protection-prod || true
                docker rm personal-data-protection-prod || true
                docker run -d -p 8080:80 --name personal-data-protection-prod ${DOCKER_IMAGE}
                """
            }
        }

        // Serve HTML on localhost:8080
        stage('Serve HTML on Localhost') {
            steps {
                echo 'Serving the HTML file on localhost:8080...'
                sh """
                echo 'Your application is now accessible at http://localhost:8080'
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            node {
                cleanWs()  // Ensuring cleanWs() is inside a node block
            }
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
