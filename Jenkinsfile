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
              // Ruta del directorio al que deseas cambiarte
              def targetDirectory = manifestFolderPath
              // Cambiarse al directorio especificado
              dir(targetDirectory) {
                sh " ls -ltr"
                sh " pwd"
                def manifestFiles = findFiles(glob: "**/*.yaml")
                echo "Cantidad de archivos encontrados: ${manifestFiles.size()} en ${targetDirectory}"
                
                if (manifestFiles.size() == 0) {
                  error("No se encontraron archivos YAML en ${manifestFolderPath}")
                }
                
                manifestFiles.each { file ->
                  //openshift.apply("--force", readFile(file: file))
                  sh "oc apply --force -f ${file}"
                  //openshift.apply("--force", file)
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
}
def getManifestFolderPath(appName) {
  return "/tmp/workspace/poc-santander/poc-santander-poc-pipeline-sync/${appName}"
}
