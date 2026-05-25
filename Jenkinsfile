pipeline {

    agent any

    tools {
        jdk 'java11'
        maven 'maven'
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {

        booleanParam(
            name: 'SKIP_SONAR',
            defaultValue: false,
            description: 'Skip SonarQube analysis'
        )

        booleanParam(
            name: 'SKIP_JACOCO',
            defaultValue: false,
            description: 'Skip JaCoCo coverage analysis'
        )

        booleanParam(
            name: 'SKIP_CHECKSTYLE',
            defaultValue: false,
            description: 'Skip Checkstyle analysis'
        )
    }

    environment {

        SONARQUBE_ENV = 'sonarqube'
    }

    stages {

        stage('Code Checkout') {

            steps {

                git branch: 'master',
                    url: 'https://github.com/itzBonCar/spring3hibernate.git'
            }
        }

        stage('Parallel Scans') {

            parallel {

                stage('Code Stability') {

                    steps {

                        sh 'mvn clean compile test'
                    }

                    post {

                        success {
                            echo 'Build stability successful'
                        }

                        failure {
                            echo 'Build stability failed'
                        }
                    }
                }

                stage('Checkstyle Analysis') {

                    when {
                        expression {
                            return !params.SKIP_CHECKSTYLE
                        }
                    }

                    steps {

                        sh 'mvn checkstyle:check'
                    }

                    post {

                        always {

                            recordIssues(
                                tools: [checkStyle(pattern: '**/checkstyle-result.xml')]
                            )
                        }
                    }
                }

                stage('JaCoCo Coverage') {

                    when {
                        expression {
                            return !params.SKIP_JACOCO
                        }
                    }

                    steps {

                        sh 'mvn clean test jacoco:report'
                    }

                    post {

                        always {

                            publishHTML([
                                reportDir: 'target/site/jacoco',
                                reportFiles: 'index.html',
                                reportName: 'JaCoCo Report',
                                keepAll: true,
                                alwaysLinkToLastBuild: true,
                                allowMissing: false
                            ])
                        }
                    }
                }

                stage('SonarQube Analysis') {

                    when {
                        expression {
                            return !params.SKIP_SONAR
                        }
                    }

                    steps {

                        withSonarQubeEnv('sonarqube') {

                            sh '''
                                mvn sonar:sonar \
                                -Dsonar.projectKey=spring3hibernate \
                                -Dsonar.projectName=spring3hibernate
                            '''
                        }
                    }
                }
            }
        }

        stage('Package Artifact') {

            steps {

                sh 'mvn clean package -DskipTests'

                archiveArtifacts(
                    artifacts: 'target/*.war',
                    fingerprint: true
                )
            }
        }

        stage('Approval Before Publish') {

            steps {

                timeout(time: 2, unit: 'MINUTES') {

                    input(
                        message: 'Approve artifact publication?',
                        ok: 'Publish'
                    )
                }
            }
        }

        stage('Publish Artifact') {

            steps {

                sh '''
                    mkdir -p published
                    cp target/*.war published/
                '''

                echo 'Artifact published successfully'
            }
        }
    }

    post {

        success {

            emailext(
                to: '92singh.yuvraj@gmail.com',
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build completed successfully.

                Job Name: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}

                Check Jenkins dashboard for reports.
                """
            )

            slackSend(
                channel: '#all-supercarbon',
                color: 'good',
                message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }

        failure {

            emailext(
                to: '92singh.yuvraj@gmail.com',
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build failed.

                Job Name: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}

                Check console logs for details.
                """
            )

            slackSend(
                channel: '#all-supercarbon',
                color: 'danger',
                message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }

        aborted {

            emailext(
                to: '92singh.yuvraj@gmail.com',
                subject: "ABORTED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: 'Pipeline was aborted during approval stage.'
            )
        }
    }
}
