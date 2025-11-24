pipeline {
    agent {
        label AGENT-1
    }
    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = ''
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    // Build
    stages {
        stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Package version: ${appVersion}"
                }
            }
        }
        stage('Install dependencies') {
            steps {
                script {
                    sh """
                        npm install
                    """
                }
            }
        }
        stage('Unit testing') {
            steps {
                script {
                    sh """
                        echo unit testing
                    """
                }
            }
        }
        // copy the steps from aws ECR
        // stage('Push image to ECR') {
        //     steps {
        //         script {
        //             withAWS(credentials: 'aws-creds', region: 'us-east-1')
        //             sh """
                    
        //             """
        //         }
        //     }
        // }

    }

}