pipeline {

    agent any

    tools {
        maven 'maven-integration'
    }

    environment {
        APP_NAME = 'tomcat-app01'
        CONTEXT_PATH = '/declarativejob'

        NEXUS_URL = 'http://3.95.22.37:8081'
        NEXUS_CREDENTIALS = 'nexus-creid'

        TOMCAT_URL = 'http://35.169.107.79:8080'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Zeeshancloud15/Tomcat_app01.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Verify WAR') {
            steps {
                sh '''
                    ls -lh target/
                    test -f target/tomcat-app01.war
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        /opt/sonar-scanner/bin/sonar-scanner
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '3.95.22.37:8081',
                    groupId: 'com.zeeshan',
                    version: '1.0',
                    repository: 'maven-release',
                    credentialsId: 'nexus-creid',
                    artifacts: [
                        [
                            artifactId: 'tomcat-app01',
                            classifier: '',
                            file: 'target/tomcat-app01.war',
                            type: 'war'
                        ]
                    ]
                )
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'tomcat-jenkins',
                        path: '',
                        url: "${TOMCAT_URL}"
                    )
                ],
                contextPath: "${CONTEXT_PATH}",
                war: 'target/tomcat-app01.war'
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully.'

            echo "Application: ${TOMCAT_URL}${CONTEXT_PATH}/"

            slackSend(
                color: 'good',
                message: """
SUCCESS: Jenkins Pipeline

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: SUCCESS

Application:
${TOMCAT_URL}${CONTEXT_PATH}/

Build URL:
${env.BUILD_URL}
"""
            )
        }

        failure {
            echo 'Pipeline failed.'

            slackSend(
                color: 'danger',
                message: """
FAILED: Jenkins Pipeline

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: FAILED

Build URL:
${env.BUILD_URL}
"""
            )
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
