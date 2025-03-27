def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
	agent any
    environment {
        SLACK_CHANNEL = '#ci-pipelines-notifications'
        SLACK_CREDENTIALS_ID = 'slacktoken' 
        NEXUS_REPO_URL = 'http://190.31.20.42:8081'
        ARTIFACT_VERSION = 'v2'
        ARTIFACT_NAME = "vprofile-${ARTIFACT_VERSION}.war"  
        AWS_S3_BUCKET = 'awscicdartifacttest'  
        AWS_BEANSTALK_APP = 'tf-test-name'
        AWS_BEANSTALK_ENV = 'vprofile-beanstalk-conf'
        AWS_REGION = 'us-east-1'  
        registryCredential = 'ecr:us-east-1:AWS-ECR-USER'
        ECR_REPO = "084828572941.dkr.ecr.us-east-1.amazonaws.com"
        imageNameURI = "vproapp-task-name"
        vprofileRegistry = "https://084828572941.dkr.ecr.us-east-1.amazonaws.com"
        cluster = "vprofile-app-ecs-cluster"
        service = "vprofile-app-ecs-service"
        IMAGE_TAG = "latest"
        COMPOSE_FILE = "compose.yaml"
        AWS_ACCOUNT_ID = "084828572941"
        AWS_DEFAULT_REGION = "us-east-1"
    }
	tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }
    stages{
        // stage('BUILD') {
        //     steps {
        //         sh 'mvn clean install -DskipTests'
        //     }
        //     post {
        //         success {
        //             echo 'Now Archiving...'
        //                 archiveArtifacts artifacts: '**/target/*.war'
        //         }
        //     }
        // }
        // stage('UNIT TEST') {
        //     steps {
        //         sh 'mvn test'
        //     }
        // }
        // stage('INTEGRATION TEST') {
        //     steps {
        //         sh 'mvn verify -DskipUnitTests'
        //     }
        // }
        // stage('CODE ANALYSIS WITH CHECKSTYLE'){
        //     steps {
        //         sh 'mvn checkstyle:checkstyle'
        //     }
        //     post {
        //         success {
        //             echo 'Generated Analysis Result'
        //         }
        //     }
        // }
        // stage('CODE ANALYSIS with SONARQUBE') {
        //     environment {
        //         scannerHome = tool 'sonar6.2'
        //     }
        //     steps {
        //         withSonarQubeEnv('Jenkins2Sonar'){
        //         sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
        //            -Dsonar.projectName=vprofile-repo \
        //            -Dsonar.projectVersion=1.0 \
        //            -Dsonar.sources=src/ \
        //            -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
        //            -Dsonar.junit.reportsPath=target/surefire-reports/ \
        //            -Dsonar.jacoco.reportsPath=target/jacoco.exec \
        //            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
        //         }
        //     }
        // }
        // stage('Quality Gate') {
        //     steps {
        //       timeout(time: 1, unit: 'HOURS') {
        //         waitForQualityGate abortPipeline: true
        //       }
        //     }
        // }
        stage('Build App Image using docker engine') {
            steps {
                script {
                    sh "docker-compose -f $COMPOSE_FILE build"
                    sh "docker tag vprofile-business-register-app-image $ECR_REPO/vprofile-business-register-app-image:$IMAGE_TAG"
                    sh "docker tag vprofile-business-register-web-image $ECR_REPO/vprofile-business-register-web-image:$IMAGE_TAG"
                    sh "docker tag vprofile-business-register-db-image $ECR_REPO/vprofile-business-register-db-image:$IMAGE_TAG"
                    sh "docker tag vprofile-business-register-mc-image $ECR_REPO/vprofile-business-register-mc-image:$IMAGE_TAG"
                }
            }
        }
        stage('login to AWS ECR'){
            steps {
                script {
                    sh "aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username jenkins --password-stdin $ECR_REPO"
                }
            }
        }
        stage('Upload App Image to AWS ECR') {
            steps {
                script {
//                    withAWS(credentials: 'AWS-ECR-USER', region: 'us-east-1'){
//                        sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username jenkins --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
//                        sh "docker push $ECR_REPO/vprofile-business-register-app-image:$IMAGE_TAG"
//                        sh "docker push $ECR_REPO/vprofile-business-register-web-image:$IMAGE_TAG"
//                        sh "docker push $ECR_REPO/vprofile-business-register-db-image:$IMAGE_TAG"
//                        sh "docker push $ECR_REPO/vprofile-business-register-mc-image:$IMAGE_TAG"
                    docker.withRegistry (vprofileRegistry, registryCredential) {
                        dockerImage.push("$ECR_REPO/vprofile-business-register-app-image:$IMAGE_TAG")
                        dockerImage.push("$ECR_REPO/vprofile-business-register-web-image:$IMAGE_TAG")
                        dockerImage.push("$ECR_REPO/vprofile-business-register-db-image:$IMAGE_TAG")
                        dockerImage.push("$ECR_REPO/vprofile-business-register-mc-image:$IMAGE_TAG")
                    }

//                    }   
                    }
                }
        }
        stage('Deploy container to ECS')  {
            steps {
                script {
                    withAWS(credentials: 'AWS-ECR-USER', region: 'us-east-1'){
                        sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                    }
                }
            }
        }

        stage('Remove images from jenkins server') {
            steps {
                script {
                   sh 'docker rmi -f $(docker images -a -q)'
                }
            }
        }
        // stage('Remove git clone file from docker stage') {
        //     steps {
        //         script {
        //             sh 'rm -rf ./vprofile-project'
        //         }
        //     }
        // }
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

        
    


