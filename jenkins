pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
    }
    stages {
        stage('CLONE CODE FROM GITHUB') {
            steps {
                git url: "https://github.com/rohitbhardwajj/TaskVault.git", branch: "main"
            }
        }
        
        stage('Install Dependencies') {
            steps {
                // Frontend
                sh 'npm install'

                // Backend
                dir('backend') {
                    sh 'npm install'
                }
            }
        }

        stage('SonarQube Quality Analysis') {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=TaskVault -Dsonar.projectKey=TaskVault -Dsonar.sources=."
                }
            }
        }

        stage('Sonar Quality Gate Scan') {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                // Frontend
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'dc'
                // Backend
                dir('backend') {
                    dependencyCheck additionalArguments: '--scan .', odcInstallation: 'dc'
                }
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
    }
}
