def org = 'djd-org'
def space = 'development'
def blue = 'demo'
def green = 'djd2'

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
}
checkpoint 'Archived'
node('git'){
   stage('Deploy') {
      wrap([$class: 'CloudFoundryCliBuildWrapper',
      apiEndpoint: 'https://api.run.pivotal.io',
      skipSslValidation: false,
      cloudFoundryCliVersion: 'cfcli',
      credentialsId: 'David-PWS',
      organization: org,
      space: space]) {

       sh "cf push '${blue}' -m 768M -i 1 -p target/articulate*jar --no-route"
       sh "cf map-route '${blue}' cfapps.io -n '${green}'"

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
      organization: org,
      space: space]) {

       sh "cf stop '${green}'"

      }
   }
}
