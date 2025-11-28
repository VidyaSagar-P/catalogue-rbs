pipeline {
    agent {
        label 'AGENT-1'
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
        // stage('Sonar Scan') {
        //     environment {
        //         scannerHome = tool 'sonar-7.2'
        //     }
        //     steps {
        //         script {
        //            // Sonar Server envrionment
        //            withSonarQubeEnv(installationName: 'sonar-7.2') {
        //                  sh "${scannerHome}/bin/sonar-scanner"
        //            }
        //         }
        //     }
        // }
        // stage('Quality Gate') {
        //     steps {
        //         timeout(time: 5, unit: 'MINUTES') {
        //             script {
        //                 def qg = waitForQualityGate()
        //                 if (qg.status != 'OK') {
        //                     error "Pipeline aborted because Quality Gate failed: ${qg.status}"
        //                 }
        //             }
        //         }
        //     }
        // }
        stage('Check Dependabot Alerts') {
            environment { 
                GITHUB_TOKEN = credentials('github-token')
            }
            steps {
                script {
                    // Fetch alerts from GitHub
                    def response = sh(
                        script: """
                            curl -s -H "Accept: application/vnd.github+json" \
                                 -H "Authorization: token ${GITHUB_TOKEN}" \
                                 https://api.github.com/repos/VidyaSagar-P/catalogue-rbs/dependabot/alerts
                        """,
                        returnStdout: true
                    ).trim()

                    // Parse JSON
                    def json = readJSON text: response

                    // Filter alerts by severity
                    def criticalOrHigh = json.findAll { alert ->
                        def severity = alert?.security_advisory?.severity?.toLowerCase()
                        def state = alert?.state?.toLowerCase()
                        return (state == "open" && (severity == "critical" || severity == "high"))
                    }

                    if (criticalOrHigh.size() > 0) {
                        error "❌ Found ${criticalOrHigh.size()} HIGH/CRITICAL Dependabot alerts. Failing pipeline!"
                    } else {
                        echo "✅ No HIGH/CRITICAL Dependabot alerts found."
                    }
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
    } //stages ended
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