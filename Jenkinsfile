pipeline {
    agent any 

    tools {
        // Must match the Maven tool name configured in Jenkins (Manage Jenkins → Global Tool Configuration)
        maven "maven3"
    }

    environment {
        // Nexus configuration
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.215.73.110:8081"
        NEXUS_REPOSITORY = "release"  // change if your repo name is different
        NEXUS_CREDENTIAL_ID = "nexus"
    }

    stages {
        stage("Clone Code") {
            steps {
                script {
                    // ✅ Public GitHub repo (no credentials needed)
                    git 'https://github.com/prakash6333/spring3-mvc-maven-xml-hello-world-1.git'
                }
            }
        }

        stage("Build with Maven") {
            steps {
                script {
                    // Clean and package the code, ignoring test failures if any
                    sh 'mvn -Dmaven.test.failure.ignore=true clean package'
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    // Read POM to extract metadata
                    pom = readMavenPom file: "pom.xml"

                    // Find the built artifact
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "Found artifact: ${filesByGlob[0].name}"

                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        echo "*** Uploading ${artifactPath} to Nexus repository: ${NEXUS_REPOSITORY}"

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: "${pom.version}",
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
                        error "*** File not found: ${artifactPath}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and upload to Nexus succeeded!"
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
        failure {
            echo "❌ Build failed. Check console logs for more details."
        }
    }
}
