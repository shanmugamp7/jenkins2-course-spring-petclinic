node {
    notify('Started')
		stage('build and test')
		{
			mvn clean package
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
              
	
	   
	   
	   
      stage('archive') {
	  
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'target/site/jacoco/', reportFiles: 'index.html', reportName: 'Code Coverage', reportTitles: ''])
             step([$class: 'JUnitResultArchiver', checksName: 'Tests', testResults: 'target/surefire-reports/TEST-*.xml'])
            archiveArtifacts artifacts: "target/*.?ar", followSymlinks: false
            
    }
    }
	
   node{
   try{
  stage("publish to nexus") {
   
   nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic', classifier: '', 
   file: 'target/petclinic.war', type: 'war']], 
   credentialsId: 'Nexus', groupId: 'org.springframework.samples',
   nexusUrl: 'vumxa95483dns.eastus2.cloudapp.azure.com:8081/nexus',
   nexusVersion: 'nexus2', protocol: 'http',
   repository: '100', 
   version: '4.2.6-SNAPSHOT'
   
   }   
   }catch(err) {
        notify ("Error $err")
        currentBuild.result='FAILURE' 
    
   }
   }
    def notify(status){
    emailext (
      to: "shanmugamp7@grr.la",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
