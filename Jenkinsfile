pipeline {
    agent any

    tools {
        maven "MAVEN3"
        jdk "OracleJDK11"
    }

    stages {
        stage('Get code') {
            steps {
                git branch: 'main', url: 'https://github.com/Gohan89/CI-with-Jenkins-SonarQube-and-Nexus.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo 'Archiving the artifacts now .............'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar'
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile " +
                       "-Dsonar.projectName=vprofile " +
                       "-Dsonar.projectVersion=1.0 " +
                       "-Dsonar.sources=src/ " +
                       "-Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ " +
                       "-Dsonar.junit.reportsPath=target/surefire-reports/ " +
                       "-Dsonar.jacoco.reportsPath=target/jacoco.exec " +
                       "-Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml"
                }
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload artifact to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.31.91.230:8081',
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: 'Vprofile-Repo',
                    credentialsId: 'nexuslogin',
                    artifacts: [
                        [artifactId: 'Vprofile-App',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                )
            }
        }
    }
}
