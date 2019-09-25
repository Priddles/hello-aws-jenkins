pipeline {
  agent any

  stages {
    stage('Deploy') {
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

          sshPut(remote: remote, from: 'html/', into: '.')
          sshCommand(remote: remote, sudo: true, command: 'cp -f html/* /usr/share/nginx/html')
        }
      }
    }
  }
}
