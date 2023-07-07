#! /usr/bin/env groovy

pipeline {
  parameters {
    string(name: 'projectName', description: 'Name of the project', defaultValue: 'poc-santander')
    string(name: 'appName', description: 'Name of the application', defaultValue: 'javaapp')
  }
  agent {
    label 'maven'
  }
  stages {
    stage('Deploy') {
      steps {
        echo 'Sync Deployment'
        script {
          openshift.withCluster() {
            openshift.withProject(params.projectName) {
              echo "Aplicativo a redesplegar: ${params.appName}"
              sh " ls -ltr"
              sh " pwd"
              def manifestFolderPath = getManifestFolderPath(params.appName)
              sh " ls -ltr /tmp/workspace/poc-santander/poc-santander-poc-pipeline-sync/javaapp"

              def manifestFiles = readFiles(glob: "${manifestFolderPath}/*.yaml")
              
              echo "Contenido de manifestFiles:"
              echo manifestFiles
              
              manifestFiles.each { fileContent ->
                openshift.apply(fileContent)
              }
              
              def deployment = openshift.selector("dc", params.appName)
              timeout(5) {
                openshift.selector("dc", params.appName).related('pods').untilEach(1) {
                  return (it.object().status.phase == "Running")
                }
              }
            }
          }
        }
      }
    }
  }
}
def getManifestFolderPath(appName) {
  return "/tmp/workspace/poc-santander/poc-santander-poc-pipeline-sync/${appName}"
}
