pipeline {
  agent any

  parameters {
      choice(name: 'PILL', choices: ['Blue pill', 'Red pill', 'Punch Morpheus'], description: 'Choose wisely...')
  }

  stages {
    stage('Build') {
      steps {
        echo 'Compiling...'
        sleep 2
        echo 'Packaging artifacts...'
        sleep 2
        echo 'Archiving artifacts to Jenkins...'
        sleep 1
      }
    }
    stage('Unit Tests') {
      steps {
        echo 'Running unit tests...'
        sleep 3
        echo 'Archiving test results to Jenkins...'
        sleep 1
      }
    }
    stage('Integration Tests') {
      parallel {
        stage('Backend-Only Tests') {
          stages {
            stage('Database Tests') {
              when {
                branch 'master'
              }
              steps {
                echo 'Running expensive backend-only database tests only on release versions...'
                sleep 3
              }
            }
            stage('Other Backend Tests') {
              steps {
                echo 'Running other backend-only integration tests...'
                sleep 7
              }
            }
          }
        }
        stage('Automated Firefox Tests') {
          steps {
            echo 'Running emulated Firefox browser tests...'
            sleep 3
          }
        }
        stage('Automated Chrome Tests') {
          steps {
            echo 'Running emulated Chrome browser tests...'
            sleep 5
          }
        }
      }
    }
    stage('Performance Tests') {
      when {
        branch 'master'
      }
      steps {
        echo 'Running expensive performance tests only on release versions...'
        sleep 5
      }
    }
    stage('Deploy to Staging') {
      parallel {
        stage('master') {
          when {
            branch 'master'
          }
          agent {
            label 'master'
          }
          steps {
            sh 'sudo cp -f html/* /usr/share/nginx/html'
          }
        }
        stage('dev') {
          when {
            expression { BRANCH_NAME ==~ /dev.*/ }
          }
          agent {
            label 'master'
          }
          steps {
            sh 'sudo cp -f html/* /usr/share/nginx/html/dev'
          }
        }
      }
    }
    stage('Deploy to Production') {
      input {
        message 'Finish deployment to production servers?'
        ok 'Deploy'
      }
      when {
        beforeAgent true
        environment(name: 'DEPLOY_TO_REMOTE', value: 'true')
      }
      environment {
        LAZY_KEY = credentials('HelloJenkinsLazyKey')
        REMOTE_DNS = credentials('HelloJenkinsProdDns')
      }

      steps {
        script {
          def remote = [:]
          remote.name = REMOTE_DNS
          remote.host = REMOTE_DNS
          remote.user = LAZY_KEY_USR
          remote.identityFile = LAZY_KEY
          remote.allowAnyHosts = true

          sshPut(remote: remote, from: 'html/', into: '.', failOnError: false)
          sshCommand(remote: remote, sudo: true, command: 'cp -f html/* /usr/share/nginx/html', failOnError: false)
        }
      }
    }
    stage('Determine Wisdom') {
      steps {
        script {
          if (params.PILL != 'Red pill') {
            unstable 'Did not take the red pill!'
          }
        }
      }
    }
    stage('Face Reality') {
      when {
        not { equals(expected: 'UNSTABLE', actual: currentBuild.result) }
      }
      steps {
        echo 'I know Kung Fu!'
      }
    }
  }
}
