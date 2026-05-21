pipeline {
    agent any

    tools {
        jdk 'java11'
        maven 'maven'
    }

    stages {


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

        stage('Feature Branch Special Stage') {

            when {
                branch 'feature-demo'
            }

            steps {
                echo 'THIS STAGE RUNS ONLY ON FEATURE BRANCH'

                sh '''
                    echo "Running feature branch logic..."
                    ls -lrt
                '''
            }
        }
    }

    post {
        success {
            echo 'Build Successful'
        }
    }
}
