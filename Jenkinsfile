// ServicePrincipal Password : 1iI8Q~O3pQOjDtY6U_70bQVViWdIlxB9yoR-Daci

import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=150e6d3b-22fc-46a4-a24c-ee9941f58d5f',
        'AZURE_TENANT_ID=ca70947f-2ab6-406e-a78f-a98999802ec8']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'test-resource'
      def webAppName = 'AppWebJava'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: '1iI8Q~O3pQOjDtY6U_70bQVViWdIlxB9yoR-Daci', usernameVariable: 'fb910883-8118-46e5-b8d1-55584202c18f')]) {
       sh '''
          az login --service-principal -u fb910883-8118-46e5-b8d1-55584202c18f -p 1iI8Q~O3pQOjDtY6U_70bQVViWdIlxB9yoR-Daci -t ca70947f-2ab6-406e-a78f-a98999802ec8
          az account set -s 150e6d3b-22fc-46a4-a24c-ee9941f58d5f
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
