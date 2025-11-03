node {
    // Define environment variables
    def NEXUS_VERSION = "nexus3"
    def NEXUS_PROTOCOL = "http"
    def NEXUS_URL = "54.215.73.110:8081"
    def NEXUS_REPOSITORY = "test"

    stage('Checkout Code') {
        echo "Cloning repository..."
        git branch: 'master', url: 'https://github.com/prakash6333/spring3-mvc-maven-xml-hello-world-1.git'
    }

    stage('Build with Maven') {
        echo "Building the project using Maven..."
        try {
            sh 'mvn -Dmaven.test.failure.ignore=true clean package'
        } finally {
            echo "Archiving JUnit test results..."
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
            }
        }
    }

    stage('Publish Artifact to Nexus') {
        echo "Preparing to upload artifacts to Nexus..."
        def pom = readMavenPom file: "pom.xml"
        def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
        echo "Found artifact: ${filesByGlob[0].name} at ${filesByGlob[0].path}"

        def artifactPath = filesByGlob[0].path
        def artifactExists = fileExists artifactPath

        if (artifactExists) {
            echo "Uploading artifact ${artifactPath} to Nexus..."
            nexusArtifactUploader(
                nexusVersion: NEXUS_VERSION,
                protocol: NEXUS_PROTOCOL,
                nexusUrl: NEXUS_URL,
                groupId: pom.groupId,
                version: "${BUILD_NUMBER}",
                repository: NEXUS_REPOSITORY,
                // credentialsId REMOVED (no auth for public repo)
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
            error "Artifact not found at ${artifactPath}"
        }
    }

    stage('Post Build Actions') {
        echo "Archiving build artifacts..."
        archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        echo "✅ Build and deployment successful. Artifact uploaded to Nexus at ${NEXUS_URL}"
    }
}
