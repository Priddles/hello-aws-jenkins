pipeline {
  agent any

  stages {
    stage('Deploy') {
      parallel {
        stage('Deploy to remote') {
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
        stage('Deploy to master') {
          agent {
            label 'master'
          }
          steps {
            sh 'sudo cp -f html/* /usr/share/nginx/html'
          }
        }
      }
    }
  }
}
