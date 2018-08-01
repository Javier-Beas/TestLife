pipeline {
    agent { label 'docker' }
    triggers {
        pollSCM ('*/2 * * * 1-5') 
    }
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
                sh "git add pom.xml && git commit -m 'changed version to -SNAPSHOT' && git push --set-upstream origin ${GIT_BRANCH}"
              }
            }
          }
        }
        stage('version fix for release branches') {
          when {
              branch "release/*" 
          }

          steps {
            script {
              def v = version()
              def branchVersion = branchVersion()
              echo "La version es ${v}, la del branch es ${branchVersion}"
              echo "Test feature 2 mroe"
              if (v != branchVersion) {
                sh "mvn versions:set -DnewVersion=${branchVersion} -DgenerateBackupPoms=false"
sh('git remote -v')
sh('git show-ref')
                sh "git add pom.xml && git commit -m 'changed version to ${branchVersion}' && git config user.email 'Jenkins@mirai.com' && git config user.name 'Jenkins' && git push --set-upstream origin ${GIT_BRANCH}"
              }

            }
          }
        }
        stage('compile') {
          steps {
            script {
              echo "branch *${GIT_BRANCH}* current build number: *${BUILD_NUMBER}*"
              echo "Commit: *${GIT_COMMIT}*"
              echo "From Pull de Github"
              currentBuild.displayName = GIT_BRANCH + "#"+ BUILD_NUMBER
              sh "mvn clean deploy -Dmaven.javadoc.skip=true -DskipTests"   
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
    def matcher = GIT_BRANCH =~ 'release/(.*)$'
    return matcher ? matcher[0][1] : null
}
