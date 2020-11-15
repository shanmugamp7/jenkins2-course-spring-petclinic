pipeline {
    agent {
        label "master"
    }
    tools {
        maven "Maven"
    }
    environment {
        NEXUS_VERSION = "nexus2"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "vumxa95483dns.eastus2.cloudapp.azure.com:8081/nexus"
        NEXUS_REPOSITORY = "100"
        NEXUS_CREDENTIAL_ID = "Nexus"
    }
		

    stages {
	notify('Started')
        
        stage("Maven Build and test") {
            steps {
                script {
                    sh "mvn clean package"
                }
            }
        }
		   stage('Publish Test results') {
	  
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'target/site/jacoco/', reportFiles: 'index.html', reportName: 'Code Coverage', reportTitles: ''])
             step([$class: 'JUnitResultArchiver', checksName: 'Tests', testResults: 'target/surefire-reports/TEST-*.xml'])
            
    }
	
	stage('Code Quality Check via SonarQube') {

        
       def scannerHome = tool 'sonar scanner';
           withSonarQubeEnv("Sonarqube") {
           sh "${tool("sonar scanner")}/bin/sonar-scanner \
           -Dsonar.projectKey=test-pet-clinic \
          -Dsonar.sources=src/main/java \
            -Dsonar.language=java \
            -Dsonar.java.binaries=target/classes \
            -Dsonar.host.url=http://vumxa95483dns.eastus2.cloudapp.azure.com \
           -Dsonar.login=19aa88bffafcb93811fa84f5e096df98f171beaa"
               }
}
        stage("Quality Gate"){
          timeout(time: 1, unit: 'MINUTES') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
        }
              
			  
	
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
	void notify(status){
    emailext (
      to: "shanmugamp7@grr.la",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )

}
}