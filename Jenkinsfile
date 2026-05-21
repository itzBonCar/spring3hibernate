pipeline {
    agent any

    tools {
        jdk 'java11'
        maven 'maven'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Branch Info') {
            steps {
                echo "Running on branch: ${env.BRANCH_NAME}"
            }
        }
    }

    post {
        success {
            echo 'Build Successful'
        }
    }
}
