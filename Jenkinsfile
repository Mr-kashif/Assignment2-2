pipeline {
    agent any

    // Triggers: Poll SCM every minute (H/2 means every 2 mins, * * * * * is every min)
    // You can also use githubPush() if configured.
    triggers {
        pollSCM('H/2 * * * *') 
    }

    environment {
        // Define recipients
        TEACHER_EMAIL = "qasimalik@gmail.com"
        MY_EMAIL = "kashig9876@gmail.com"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Deploy') {
            steps {
                script {
                    echo 'Stopping old containers...'
                    // -v removes volumes (dev dependencies might be large, but we mount local folders so it is fine)
                    sh 'docker-compose down || true'

                    echo 'Starting build environment...'
                    // We use the same docker-compose for "Building" in this context as it installs npm packages live
                    sh 'docker-compose up -d'
                }
            }
        }

        stage('Wait for Build') {
            steps {
                script {
                    echo 'Waiting for npm install and start...'
                    sleep 60
                }
            }
        }
    }

    post {
        success {
            echo 'Build Environment Up!'
            emailext subject: "Jenkins Build Success: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                     body: "The application build environment is successfully up and running.\n\nProject: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}",
                     to: "${env.TEACHER_EMAIL}, ${env.MY_EMAIL}"
        }
        failure {
            echo 'Build Failed!'
            emailext subject: "Jenkins Build Failed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                     body: "The build failed.\n\nProject: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}",
                     to: "${env.TEACHER_EMAIL}, ${env.MY_EMAIL}"
        }
    }
}
