#! /usr/bin/env groovy

parameters {
  string(name: 'projectName', description: 'Name of the project', defaultValue: 'poc-santander')
  string(name: 'appName', description: 'Name of the application', defaultValue: 'javaapp')
}

pipeline {
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
              sh 'ls -ltr'
              
              def manifestFolderPath = getManifestFolderPath(params.appName)
              def manifestFiles = findManifestFiles(manifestFolderPath)
              
              manifestFiles.each { file ->
                openshift.apply("--force", readFile(file: file))
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
  return "manifests/${appName}"
}

def findManifestFiles(folderPath) {
  def manifestFiles = findFiles(glob: "${folderPath}/*.yaml")
  if (manifestFiles.empty) {
    throw new RuntimeException("No YAML files found in ${folderPath}")
  }
  return manifestFiles.collect { it.path }
}
