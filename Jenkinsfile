def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
	agent any
    environment {
        SLACK_CHANNEL = '#ci-pipelines-notifications'  // Change to your Slack channel
        SLACK_CREDENTIALS_ID = 'slacktoken'  // Add your Slack token in Jenkins credentials
        NEXUS_REPO_URL = 'http://13.218.32.114:8081'
        ARTIFACT_VERSION = 'v2'
        ARTIFACT_NAME = "vprofile-${ARTIFACT_VERSION}.war"  // Change this to match your artifact
        AWS_S3_BUCKET = 'awscicdartifacttest'  // S3 bucket for deployment
        AWS_BEANSTALK_APP = 'tf-test-name'
        AWS_BEANSTALK_ENV = 'vprofile-beanstalk-conf'
        AWS_REGION = 'us-east-1'  // Change to your AWS region
    }
	tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }
    stages{
        stage('BUILD') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                        archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('UNIT TEST') {
            steps {
                            sh 'mvn test'
            }
        }
        stage('INTEGRATION TEST') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('CODE ANALYSIS with SONARQUBE') {
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
                withSonarQubeEnv('Jenkins2Sonar'){
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: '172.31.20.42:8081',
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: 'vprofile-repo',
                  credentialsId: 'nexuslogin',
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: "target/${ARTIFACT_NAME}",
                     type: 'war']
                  ]
                )
            }
        }
        stage('Copy Artifact to AWS S3 Bucket') {
            steps {
                def artifactUrl = "${env.NEXUS_REPO_URL}/${env.ARTIFACT_NAME}"
                echo "Downloading artifact from Nexus: ${artifactUrl}"
                withAWS(credentials: '6dc5080f-6a4d-4e5d-88ee-ce8477629d20', region: "{AWS_REGION}") {
                    sh """
                    wget -O ${env.ARTIFACT_NAME} ${artifactUrl}
                    aws s3 cp ${env.ARTIFACT_NAME} s3://${env.AWS_S3_BUCKET}/
                    """
                }
            }
        }
        stage('Deploy to AWS Elastic Beanstalk') {
            steps {
                echo "Deploying artifact to AWS Elastic Beanstalk..."
                withAWS(credentials: '6dc5080f-6a4d-4e5d-88ee-ce8477629d20', region: "{AWS_REGION}") {
                    sh """
                    aws elasticbeanstalk create-application-version --application-name ${env.AWS_BEANSTALK_APP} --version-label build-${env.BUILD_NUMBER} --source-bundle S3Bucket=${env.AWS_S3_BUCKET},S3Key=${env.ARTIFACT_NAME} --region ${env.AWS_REGION}
                    aws elasticbeanstalk update-environment --application-name ${env.AWS_BEANSTALK_APP} --environment-name ${env.AWS_BEANSTALK_ENV} --version-label build-${env.BUILD_NUMBER} --region ${env.AWS_REGION}
                    """
                }  
            }
        }
    }
    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def buildStatus = currentBuild.currentResult // SUCCESS, FAILURE, UNSTABLE, etc.
                def testResults = "Tests completed: ${buildStatus}" // Modify based on actual test results

                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: buildStatus == 'SUCCESS' ? 'good' : 'danger',
                    message: """
                    *Job:* ${jobName} #${buildNumber}
                    *Status:* ${buildStatus}
                    *Test Results:* ${testResults}
                    *Nexus Repository:* <${env.NEXUS_REPO_URL}|View Artifact>
                    """
                )
            }
        }
    }
}
            // echo 'Slack Notifications.'
            // slackSend channel: '${}',
            //     color: COLOR_MAP[currentBuild.currentResult],
            //     message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        
    


