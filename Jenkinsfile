pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "jessicak/myapp:%BUILD_NUMBER%"
        SONARQUBE = 'SonarQube'  // SonarQube server name in Jenkins
    }

    stages {

        stage('Build') {
            steps {
                echo 'Building the application...'
                // Maven build
                bat '.\\mvnw.cmd clean package -DskipTests'
                // Build Docker image
                bat "docker build -t %DOCKER_IMAGE% ."
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                bat '.\\mvnw.cmd test'
                junit '**\\target\\surefire-reports\\*.xml'
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv(SONARQUBE) {
                    bat '.\\mvnw.cmd sonar:sonar'
                }
            }
        }

        stage('Security') {
            steps {
                echo 'Running OWASP Dependency Check...'
                bat '.\\mvnw.cmd org.owasp:dependency-check-maven:check'
                // Optionally parse report to fail build if critical vulnerabilities found
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to staging environment...'
                // Run Docker container
                bat "docker run -d -p 8080:8080 --name myapp-staging %DOCKER_IMAGE%"
            }
        }

        stage('Release') {
            steps {
                input message: 'Approve release to production?'
                echo 'Deploying to production...'
                bat "docker tag %DOCKER_IMAGE% jessicak/myapp:latest"
                bat "docker push jessicak/myapp:latest"
            }
        }

        stage('Monitoring') {
            steps {
                echo 'Monitoring application...'
                // Simple health check
                bat 'powershell -Command "try { Invoke-WebRequest -Uri http://localhost:8080/actuator/health -UseBasicParsing -ErrorAction Stop } catch { Write-Host \'Alert: App is down!\' }"'
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
