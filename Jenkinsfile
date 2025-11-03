pipeline {
    agent any
    tools {
        maven 'maven3'
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.133.145.136:8081"
        NEXUS_REPOSITORY = "devops"
        NEXUS_CREDENTIAL_ID = "Nexus_server"
    }

    stages {

        stage("Clone Code") {
            steps {
                script {
                    git url: 'https://github.com/prakash6333/spring3-mvc-maven-xml-hello-world-1.git', branch: 'master'
                }
            }
        }

        stage("Build with Maven") {
            steps {
                script {
                    sh 'mvn -Dmaven.test.failure.ignore=true clean package'
                }
            }
            post {
                always {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                    }
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "Found artifact: ${filesByGlob[0].name} at ${filesByGlob[0].path}"

                    def artifactPath = filesByGlob[0].path
                    def artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        echo "Uploading artifact: ${artifactPath}, Group: ${pom.groupId}, Version: ${BUILD_NUMBER}"

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: "${BUILD_NUMBER}",
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging
                                ],
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"
                                ]
                            ]
                        )
                    } else {
                        error "File not found: ${artifactPath}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment successful. Artifact uploaded to Nexus at ${NEXUS_URL}"
            archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        }
        failure {
            echo "Build failed. Please check logs for details."
        }
    }
}
