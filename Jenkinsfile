properties properties: [
        [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '10']],
        disableConcurrentBuilds()
]

timeout(60) {
  node('nativescript') {
      def buildNumber = env.BUILD_NUMBER
      def branchName = env.BRANCH_NAME
      def workspace = env.WORKSPACE
      def buildUrl = env.BUILD_URL

      // PRINT ENVIRONMENT TO JOB
      echo "workspace directory is $workspace"
      echo "build URL is $buildUrl"
      echo "build Number is $buildNumber"
      echo "branch name is $branchName"
      echo "PATH is $env.PATH"

      try {
          stage('Checkout') {
              checkout scm
          }

          stage('Build') {
              sh "npm run clean && npm run build"
          }

          stage('Test') {
              sh "npm run test"
              //junit 'target/surefire-reports/unit/*/TEST-*.xml'
          }

          stage('E2E') {
              sh "npm run e2e"
              junit 'target/surefire-reports/NativeScript_/*/TEST-*.xml'
          }

          stage('Publish NPM snapshot') {
              def currentVersion = sh(returnStdout: true, script: "npm version | grep \"{\" | tr -s ':'  | tr '_' '-' | tr '/' '-' | cut -d \"'\" -f 4").trim()
              def newVersion = "${currentVersion}-${branchName}-${buildNumber}"
              sh "npm version ${newVersion} --no-git-tag-version && npm publish --tag next"
          }

      } catch (e) {
          mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}): Error on build", to: 'github@martinreinhardt-online.de', body: "Please go to ${env.BUILD_URL}."
          throw e
      }
  }
}

