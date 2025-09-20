pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "jessicak/myapp:%BUILD_NUMBER%"
        SONARQUBE = 'MySonarServer'  // SonarQube server name in Jenkins
    }

    stages {

        stage('Build') {
            steps {
                echo 'Building the application...'
                // Build using installed Maven
                bat 'mvn clean package -DskipTests'
                // Build Docker image
                bat "docker build -t %DOCKER_IMAGE% ."
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                bat 'mvn test'
                junit '**\\target\\surefire-reports\\*.xml'
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv(SONARQUBE) {
                    bat 'mvn sonar:sonar'
                }
            }
        }

        stage('Security') {
            steps {
                echo 'Running OWASP Dependency Check...'
                bat 'mvn org.owasp:dependency-check-maven:check'
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
                // Simple health check using PowerShell
                bat 'powershell -Command "try { Invoke-WebRequest -Uri http://localhost:8080/actuator/health -UseBasicParsing -ErrorAction Stop } catch { Write-Host \'Alert: App is down!\' }"'
            }
        }
    }

    post {
        failure {
            mail to: 'jessikakshapati@gmail.com',
                 subject: "Build failed in Jenkins: ${currentBuild.fullDisplayName}",
                 body: "Check Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
