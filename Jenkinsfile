pipeline {
	agent any
	tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.40.209:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	    NEXUS_REPO_ID    = "vprofile-release"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
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
	stage('UNIT TEST'){
        steps {
            sh 'mvn test'
        }
    }
	stage('INTEGRATION TEST'){
        steps {
            sh 'mvn verify -DskipUnitTests'
        }
    }
    stage ('CODE ANALYSIS WITH CHECKSTYLE'){
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
            withSonarQubeEnv('sonarserver'){
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
}
