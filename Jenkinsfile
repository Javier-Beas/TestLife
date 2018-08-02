pipeline {
    agent { label 'docker' }
    stages {
        stage('version fix for develop branch') {
          when {
              branch "develop"
          }
          steps {
            script {
              def v = version()
              echo "La versión en el pom de develop es ${v}"
              if (!v.endsWith("-SNAPSHOT")) {
                echo "No es un snapshot, añadimos -SNAPSHOT a la versión"
                sh "mvn versions:set -DnewVersion=${v}-SNAPSHOT -DgenerateBackupPoms=false"
                sshagent(credentials: ['c4ba2de8-d7d5-4a1d-8c7c-7369c21c027a']) {
                  sh "git add pom.xml && git commit -m 'changed version to ${v}-SNAPSHOT' && git push --set-upstream origin ${GIT_BRANCH}"
                }
              }
            }
          }
        }
        stage('version fix for release branches') {
          when {
              branch "(release.*|hotfix.*)" 
          }

          steps {
            script {
              def v = version()
              def branchVersion = branchVersion()
              echo "La version es ${v}, la del branch es ${branchVersion}"
              echo "Test feature 2 mroe"
              if (v != branchVersion) {
                sh "mvn versions:set -DnewVersion=${branchVersion} -DgenerateBackupPoms=false"
                sshagent(credentials: ['c4ba2de8-d7d5-4a1d-8c7c-7369c21c027a']) {
                  sh "git add pom.xml && git commit -m 'changed version to ${branchVersion}-SNAPSHOT' && git push --set-upstream origin ${GIT_BRANCH}"
                }
              }

            }
          }
        }
        stage('compile') {
          steps {
            script {
              echo "branch *${GIT_BRANCH}* current build number: *${BUILD_NUMBER}*"
              echo "Commit: *${GIT_COMMIT}*"
              echo "From Pull de Github v0.0.4"
              currentBuild.displayName = GIT_BRANCH + "#"+ BUILD_NUMBER
              sshagent(credentials: ['c4ba2de8-d7d5-4a1d-8c7c-7369c21c027a']) {
                sh "mvn clean deploy -Dmaven.javadoc.skip=true -DskipTests"
              }   
            }
          }
       }
    } 
 }
 
 def version() {
    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
    return matcher ? matcher[0][1] : null
}
def branchVersion () {
    def matcher = GIT_BRANCH =~ '(?:release|hotfix)/(.*)$'
    return matcher ? matcher[0][1] : null
}
