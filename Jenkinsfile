pipeline {
    agent {
        label AGENT-1
    }
    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '697576974187'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
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
        stage('Push image to ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com
                        docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                        docker push 697576974187.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                    
                }
            }
        }
        stage('Trigger deploy') {
            when {
                expression { params.deploy }
            }
            steps {
                script {
                    // trigger downstream job with parameters; correct argument form
                    build job: 'catalogue-cd',
                          parameters: [
                              string(name: 'appVersion', value: "${env.APP_VERSION}"),
                              string(name: 'deploy_to', value: 'dev')
                          ],
                          propagate: false,
                          wait: false
                }
            }
        }
    } // end stages
        // stage('Trigger deploy') {
        //     when{
        //         expression { params.deploy }
        //     }
        //     steps {
        //         script {
        //             build job: 'catalogue-cd'
        //                 parameters: [
        //                 string(name: 'appVesrion', value: "${appVersion}"),
        //                 string(name: 'deploy_to', value: 'dev')
        //                 ],
        //             propagate: false,  // even SG fails VPC will not be effected
        //             wait: false // VPC will not wait for SG pipeline completion
        //         }
        //     }
        // }
    // }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'Hello Success'
        }
        failure { 
            echo 'Hello Failure'
        }
    }

}