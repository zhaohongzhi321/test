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
rm -rf ./base-project/src
cp -r ./test_master ./base-project/src
cd ./base-project/
rm -rf dist
npm run build'''
      }
    }

  }
}