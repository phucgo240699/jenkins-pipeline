pipeline {
  agent any
  environment {
    FRONTEND_GIT = 'https://github.com/phucgo240699/base-reactjs-ts.git'
    FRONTEND_BRANCH = 'master'
    FRONTEND_IMAGE = 'jaywalker99/base-reactjs-ts'
    FRONTEND_SERVER = 'https://159.65.142.128'
    FRONTEND_SERVER_DIR = './app'
  }
  stages {
    stage('Build JS') {
      agent {
        docker {
          image 'node:latest'
          args '-v tutorial_jenkins_frontend_modules:$WORKSPACE/node_modules'
        }
      }
      steps {
        git(url: FRONTEND_GIT, branch: FRONTEND_BRANCH)
        sh 'npm i'
        sh 'npm run build'
        stash(name: 'frontend', includes: 'build/*/**')
      }
    }
    stage('Build Image') {
      steps {
        unstash 'frontend'
        script {
          docker.withRegistry('', 'docker-hub') {
            def image = docker.build(FRONTEND_IMAGE)
            image.push(BUILD_ID)
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          withCredentials([sshUserPrivateKey(
            credentialsId: 'ssh',
            keyFileVariable: 'identityFile',
            passphraseVariable: '',
            usernameVariable: 'user'
          )]) {
            def remote = [:]
            remote.name = 'server'
            remote.host = FRONTEND_SERVER
            remote.user = user
            remote.identityFile = identityFile
            remote.allowAnyHosts = true

            sshCommand remote: remote, command: "cd $FRONTEND_SERVER_DIR && export FRONTEND_IMAGE=$FRONTEND_IMAGE:$BUILD_ID && docker-compose up -d"
          }
        }
      }
    }
  }
}