pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }
    environment {
        SONAR_SERVER = 'sonarserver'
        SONAR_SCANNER = 'sonarscanner'
        NEXUS_URL = '18.220.9.204'
        NEXUS_PORT = '8081'
        NEXUS_REPO = 'vprofile-repo'
        NEXUS_CREDS = 'bddc5342-1a14-4114-833f-7139bb33102a'
        ARTVERSION = "${env.BUILD_ID}"
    }
    stages {
        stage('Fetch code') {
            steps {
                git branch: 'atom', url: 'https://github.com/quecyakuffo/prot.git'
            }
        }
        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo "Archiving artifact"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
                withCredentials([string(credentialsId: 'sonartoken', variable: 'SONAR_TOKEN')]) {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml \
                        -Dsonar.host.url=http://172.31.46.248:9000 \
                        -Dsonar.login=${SONAR_TOKEN}'''
                }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    withCredentials([string(credentialsId: 'sonartoken', variable: 'SONAR_TOKEN')]) {
                        waitForQualityGate abortPipeline: true, credentialsId: 'sonartoken'
                    }
                }
            }
        }
        stage('Upload Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'bddc5342-1a14-4114-833f-7139bb33102a', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                        curl -v -u ${NEXUS_USER}:${NEXUS_PASS} \
                        --upload-file target/vprofile-v2.war \
                        "http://${NEXUS_URL}:${NEXUS_PORT}/repository/${NEXUS_REPO}/QA/vproapp/${ARTVERSION}/vproapp-${ARTVERSION}.war"
                    '''
                }
            }
        }
    }
}
