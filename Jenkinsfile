pipeline {
    agent { label 'test-agent' }
    tools {
        jdk 'java11'
        maven 'maven'
    }

    parameters {
        // string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

        // text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

        // booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')

        choice(name: 'ENVIRONMENT', choices: ['dev', 'uat', 'prod'], description: 'Pick something')
    }

    stages {
        stage('checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/OT-MyGurukulam/spring3hibernate.git']])
            }
        }
        
        stage('GitLeaks Scan') {
            steps {
                sh '''
                    gitleaks detect \
                    --source . \
                    --report-format json \
                    --report-path gitleaks-report.json \
                    || true
                '''
            }
        }
        
        stage('Parallel Stage') {
            parallel {
                stage('compile') {
                    // input {
                    //     message "Should we continue?"
                    //     ok "Yes, we should."
                    //     submitter "alice,bob"
                    //     parameters {
                    //         string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
                    //     }
                    // }
                    steps {
                        sh 'mvn compile'
                    }
                    
                }
                // stage('test') {
                //     steps {
                //         sh 'mvn test'
                //     }
                // }
                stage('package') {
                    when {
                        expression { params.ENVIRONMENT == 'prod' }
                    }
                    steps {
                        sh 'mvn package -DskipTests'
                    }
                }
            }
        }   
    }
 
}
