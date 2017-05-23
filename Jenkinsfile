:q
node ('git') {
   def mvnHome
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/ddulek/pcf-articulate-code.git'
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'M3'
   }
   stage('Build') {
      // Run the maven build
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
   stage('Results') {
      archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
   }   
   stage('Deploy') {
      wrap([$class: 'CloudFoundryCliBuildWrapper',
      apiEndpoint: 'https://api.run.pivotal.io',
      skipSslValidation: false,
      cloudFoundryCliVersion: 'cfcli',
      credentialsId: 'David-PWS',
      organization: 'did-org',
      space: 'development']) {

       sh 'cf push articulate-djd -m 512M -i 1 -p target/articulate*jar --no-route'
       sh 'cf map-route articulate-djd cfapps.io -n djd2'

      }
   }
   stage('Validate') {
      input message: 'New version valid ?', ok: 'Yes'
   }
   stage('Cleanup') {
      wrap([$class: 'CloudFoundryCliBuildWrapper',
      apiEndpoint: 'https://api.run.pivotal.io',
      skipSslValidation: false,
      cloudFoundryCliVersion: 'cfcli',
      credentialsId: 'David-PWS',
      organization: 'did-org',
      space: 'development']) {

       sh 'cf stop djd2'

      }
   }
}
