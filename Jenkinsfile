pipeline {
    agent any

    tools {
        sonarqube 'SonarQube-Scanner-Latest'
    }

    environment {
        SONAR_CRED_ID = 'sonarqube-token'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '1. Pulling application code from GitHub...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '2. Installing Node.js dependencies...'
                sh 'npm ci'
            }
        }

        stage('Run Automated Tests & Get Coverage') {
            steps {
                echo '3. Running automated tests and generating coverage report...'
                sh 'npm test'
            }
        }

        stage('Static Code Analysis (ESLint)') {
            steps {
                echo '4. Performing static code checks with ESLint...'
                sh 'npm run lint'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '5. Running SonarQube analysis...'
                withSonarQubeEnv('My SonarQube') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=jenkins-exam-project \
                        -Dsonar.sources=. \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    """
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                echo '6. Checking SonarQube Quality Gate status...'
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Archive Artifacts for Release') {
            when {
                tag "release-*"
            }
            steps {
                echo '7. Archiving artifacts for a release build...'
                archiveArtifacts artifacts: 'coverage/**, app.js', followSymlinks: false
            }
        }
    }

    post {
        always {
            echo 'Sending email notification...'
            mail to: 'your-email-for-notifications@example.com',
                 subject: "Build ${currentBuild.fullDisplayName}: ${currentBuild.result}",
                 body: """Build Summary:
                 - Project: ${env.JOB_NAME}
                 - Build Number: ${env.BUILD_NUMBER}
                 - Status: ${currentBuild.result}
                 - URL: ${env.BUILD_URL}
                 """
        }
    }
}