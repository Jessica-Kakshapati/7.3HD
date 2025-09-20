pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "jessicak/my-spring-app:${env.BUILD_NUMBER}"
        SONARQUBE = 'SonarQube' // SonarQube installation name in Jenkins
    }

    stages {

        stage('Build') {
            steps {
                echo 'Building the application...'
                // Example for Maven
                sh './mvnw clean package -DskipTests'
                // Build Docker image
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh './mvnw test'
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv(SONARQUBE) {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        stage('Security') {
            steps {
                echo 'Running OWASP Dependency Check...'
                sh './mvnw org.owasp:dependency-check-maven:check'
                // Optionally parse report and fail build if critical issues found
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to staging environment...'
                sh "docker run -d -p 8080:8080 --name myapp-staging ${DOCKER_IMAGE}"
            }
        }

        stage('Release') {
            steps {
                input message: 'Approve release to production?'
                echo 'Deploying to production...'
                sh "docker tag ${DOCKER_IMAGE} yourusername/myapp:latest"
                sh "docker push yourusername/myapp:latest"
                // Optionally trigger AWS CodeDeploy or Octopus Deploy
            }
        }

        stage('Monitoring') {
            steps {
                echo 'Monitoring application...'
                // Example: simple health check
                sh 'curl -f http://localhost:8080/actuator/health || echo "Alert: App is down!"'
                // For real monitoring, integrate Datadog/NewRelic
            }
        }
    }

    post {
        failure {
            mail to: 'team@example.com',
                 subject: "Build failed in Jenkins: ${currentBuild.fullDisplayName}",
                 body: "Check Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
