properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '14', numToKeepStr: '5')), pipelineTriggers([])])

def templatePath = "./openshift/templates/httpd.json"
def templateName = "httpd-example"

pipeline {
  agent any

  options {
    timeout(time: 10, unit: 'MINUTES')
  }

  stages {
    stage('preamble'){
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              echo "Using projct: ${openshift.project()}"
            }
          }
        }
      }
    }


    stage('create') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              if (! openshift.selector('dc', templateName).exists()) {
                openshift.newApp(templatePath, "--param=SOURCE_REPOSITORY_URL=https://github.com/Madomur/httpd-ex.git")
              } else {
                openshift.selector('bc', templateName).startBuild()
              }
            }
          }
        }
      }
    }

    stage('build') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              def builds = openshift.selector('bc', templateName).related('builds')
              timeout(5) {

                builds.untilEach(1) {
                  return (it.object().status.phase == "Complete")
                }
              }
            }
          }
        }
      }
    }

    stage('deploy') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              def deploydesc = openshift.selector("dc", templateName)
              deploydesc.rollout()

              timeout(5) {
                deploydesc.related('pods').untilEach(1) {
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