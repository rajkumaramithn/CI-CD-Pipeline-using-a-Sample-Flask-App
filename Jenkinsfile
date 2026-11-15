
pipeline {
    agent any

    environment {
        MONGO_URI  = 'mongodb://localhost:27017/test_student_db'
        SECRET_KEY = 'jenkins-test-key'
    }

    stages {
        stage('Build') {
            steps {
                bat 'py -m venv venv'
                bat 'venv\\Scripts\\python -m pip install --upgrade pip'
                bat 'venv\\Scripts\\python -m pip install -r requirements.txt'
            }
        }
        stage('Test') {
            steps {
                bat 'venv\\Scripts\\python -m pytest -v'
            }
        }
        stage('Deploy to Staging') {
            steps {
                bat '''
                if not exist C:\\staging\\flask_app mkdir C:\\staging\\flask_app
                xcopy /E /I /Y /EXCLUDE:.gitignore . C:\\staging\\flask_app
                '''
                echo 'Deployed to staging: C:\\staging\\flask_app'
            }
        }
    }

    post {
        success {
            mail to: 'rajkumar.amithn@gmail.com',
                 subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build passed. ${env.BUILD_URL}"
        }
        failure {
            mail to: 'rajkumar.amithn@gmail.com',
                 subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build failed. ${env.BUILD_URL}"
        }
    }
}