pipeline {
  agent {
    label 'hydra-node'
  }
  options {
    ansiColor('xterm')
    parallelsAlwaysFailFast()
    timeout(time: 10, unit: 'MINUTES')
  }
  parameters {
    booleanParam(name: 'PUBLISH', defaultValue: false, description: 'Publish react-scripts')
  }
  stages {
    stage('Publish') {
      when {
        anyOf {
          tag 'adaptive-*'
          expression { return params.PUBLISH }
        }
      }
      environment {
        ARTIFACTORY_URL = 'https://weareadaptive.jfrog.io/artifactory'
        ARTIFACTORY = credentials('travis-hydra-artifactory')
        VERSION="""${sh(
            returnStdout: true,
            script: '''#!/bin/bash
              VERSION="${TAG_NAME:-$(git describe --tags --abbrev=0)}"
              echo -n "${VERSION##adaptive-}"
            '''
            )}""".trim()
      }
      steps {
        buildName "#${BUILD_NUMBER} (${env.VERSION})"
        configFileProvider([configFile(fileId: 'hydra-npmrc', targetLocation: 'packages/react-scripts/.npmrc')]) {
          dir('packages/react-scripts') {

            sh label: 'configure', script: '''#!/bin/bash
                  npm config set allow-same-version true
                  npm config set git-tag-version false
                  npm version "${VERSION}"'''
            sh 'npm publish'
          }
        }
      }
    }
  }
  post {
    always {
      script {
        def branch = env.CHANGE_BRANCH != null ? "(${env.CHANGE_BRANCH})" : ""
        def color = "${currentBuild.result}" == "SUCCESS" ? "#00FF00" : "#FF0000"
        def emoji = "${currentBuild.result}" == "SUCCESS" ? ":white_check_mark:" : ":fire:"
        def message = "${emoji} ${currentBuild.result}:  Job '${env.JOB_NAME} ${branch} #${env.BUILD_NUMBER}' (${env.BUILD_URL}) in ${currentBuild.durationString}"
        slackSend(color: color, message: message)
      }
    }
  }
}
