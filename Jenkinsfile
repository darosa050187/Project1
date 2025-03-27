//def COLOR_MAP = [
//    'SUCCESS': 'good', 
//    'FAILURE': 'danger',
//]
def notifySlack(String buildStatus) {
    def colorCode = buildStatus == 'SUCCESS' ? '#36a64f' : '#ff0000'
    def summary = "*Job:* ${env.JOB_NAME} #${env.BUILD_NUMBER}\n" +
                 "*Status:* ${buildStatus}\n" +
                 "*Duration:* ${currentBuild.durationString}\n" +
                 "*Details:* ${env.BUILD_URL}"

    slackSend(
        channel: env.SLACK_CHANNEL,
        color: colorCode,
        message: summary,
        tokenCredentialId: env.SLACK_CREDENTIALS_ID
    )
}

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
        registryCredential = 'ecr:us-east-1:AWS'
        ECR_REPO = "084828572941.dkr.ecr.us-east-1.amazonaws.com"
        imageNameURI = "vproapp-task-name"
        vprofileRegistry = "https://084828572941.dkr.ecr.us-east-1.amazonaws.com"
        cluster = "vprofile-app-ecs-cluster"
        service = "vprofile-app-ecs-service"
        IMAGE_TAG = "latest"
        COMPOSE_FILE = "compose.yaml"
        AWS_ACCOUNT_ID = "084828572941"
        AWS_DEFAULT_REGION = "us-east-1"
        REQUIRED_TOOLS = "docker, aws, java"
    }
//    tools {
//        maven "MAVEN3.9"
//        jdk "JDK17"
//    }
    stages{
        stage('Validate Jenkins Environment') {
            steps {
                script {
                    def tools = REQUIRED_TOOLS.split(',').collect { it.trim() }
                    echo "Checking for required tools: ${tools.join(', ')}"

                    tools.each { tool ->
                        echo "Checking for ${tool}..."
                        def exitCode = sh(script: "which ${tool}", returnStatus: true)
                        if (exitCode != 0) {
                            error "Required tool '${tool}' is not installed"
                        }
                        echo "${tool} found successfully"
                    }
                    echo "All required tools are available"
                }
            }
        }
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
        // stage('login to AWS ECR'){
        //     steps {
        //         script {
        //             try {
        //                 def ecrPassword = sh(script: "aws ecr get-login-password --region ${AWS_DEFAULT_REGION}", returnStdout: true).trim()
        //                 sh "echo \${ecrPassword} | docker login --username AWS --password-stdin ${ECR_REPO}"
        //             } catch (Exception e) {
        //                 error "Failed to login to ECR: ${e.message}"
        //             }
        //         }
        //     }
        // }
        stage('Environment Health Check') {
            steps {
                script {
                    try {
                        withAWS(credentials: 'AWS', region: 'us-east-1') {
                            sh '''aws ecs describe-services \
                                --cluster ${cluster} \
                                --services ${service} \
                                --query 'services[0].deployments[0].rolloutState' \
                                --output text '''
                        }
                    } catch (Exception e) {
                        error "Deployment health check failed: ${e.message}"
                    }
                }
            }
        }
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
        stage('Upload App Image to AWS ECR') {
            steps {
                script {
                    def images = [
                        'vprofile-business-register-app-image',
                        'vprofile-business-register-web-image',
                        'vprofile-business-register-db-image',
                        'vprofile-business-register-mc-image'
                    ]
                    docker.withRegistry (vprofileRegistry, registryCredential) {
                        images.each { imageName ->
                            try {
                                def fullImangeName = "${ECR_REPO}/${imageName}:${IMAGE_TAG}"
                                docker.image(imageName).push(fullImageName)
                            } catch (Exception e) {
                                error "Failed to push ${image_Name}: ${e.message}"
                            }
                        }
                    }
                }
            }
        }
        stage('Deploy container to ECS')  {
            steps {
                script {
                    withAWS(credentials: 'AWS', region: 'us-east-1'){
                        sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                    }
                }
            }
        }
        stage('Clean up Processes') {
            parallel {
                stage('Remove images from jenkins server') {
                    steps {
                        script {
                            try {
                                sh 'docker system -af --volumes'
                                sh 'docker rmi -f $(docker images -a -q) || true'
                            } catch (Exception e) {
                                echo "Warning: Failed to clean up Docker images: ${e.message}"
                            }
                        }
                    }
                }
                stage('Remove git clone file from docker stage') {
                    steps {
                        script {
                            sh 'rm -rf ./vprofile-project'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                if (currentBuild.result == 'SUCCESS') {
                    notifySlack('SUCCESS')
                }
                else if (currentBuild.result == 'FAILURE') {
                    notifySlack('FAILURE')
                }
                else if (currentBuild.result == 'UNSTABLE') {
                    notifySlack('UNSTABLE')
            }
            }
        }
    }
}

        
    


