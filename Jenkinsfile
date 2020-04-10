pipeline {
  agent any
  stages {
    stage('update') {
      steps {
        git(url: 'https://github.com/zhaohongzhi321/test.git', branch: 'master', changelog: true, poll: true)
      }
    }

    stage('build') {
      steps {
        sh '''cd ..
zip -r src.zip test_master'''
      }
    }

  }
}