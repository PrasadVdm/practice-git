pipeline {
    agent any

    environment {
        NEXUS_URL = '172.31.22.182:8081'  // Nexus URL
        NEXUS_REPO = 'vprofile-repo' 
        NEXUS_CRED = 'nexuslogin'              // Nexus repository name
        ARTIFACT_PATH = 'target/vprofile-v2.war'          // Path to the artifact
        GROUP_ID = 'QA'                    // Maven Group ID
        ARTIFACT_ID = 'vprofile'                       // Artifact ID
    }

    options {
        disableConcurrentBuilds()
    }

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'atom', url: 'https://github.com/PrasadVdm/vprofile-project-2025.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
        }

        stage('Archive Aftifact') {
            steps {
                archiveArtifacts artifacts: '**/*.war'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
            post {
                success {
                    sh 'echo "Hey there, Tests run successfully"'
                }
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarserver') {  // Use the configured SonarQube instance
                    script {
                        def scannerHome = tool 'sonartool'  // Use the custom tool name configured in Jenkins
                            sh """
                                ${scannerHome}/bin/sonar-scanner \
                                    -Dsonar.projectKey="vprofile-project" \
                                    -Dsonar.projectName="VDM Vprofile Project" \
                                    -Dsonar.projectVersion=1.0 \
                                    -Dsonar.sources=src/ \
                                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                                    -Dsonar.junit.reportPaths=target/surefire-reports/ \
                                    -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                            """
                        }
                }
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') { 
                    waitForQualityGate abortPipeline: true // Abort pipeline if QG fails
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3', 
                    protocol: 'http',
                    nexusUrl: "${NEXUS_URL}",
                    repository: "${NEXUS_REPO}",
                    credentialsId: "${NEXUS_CRED}",
                    groupId: "${GROUP_ID}",
                    version: "v${BUILD_ID}_${BUILD_TIMESTAMP}",
                    artifacts: [
                        [artifactId: "${ARTIFACT_ID}",
                         classifier: '',
                         file: "${ARTIFACT_PATH}",
                         type: 'war']
                    ]
                )
            }
        }
    }
}