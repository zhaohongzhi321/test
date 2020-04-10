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
cp -r ./test_master ./src
zip -r src.zip src
cp src.zip ./base-project/
cd ./base-project/
rm -rf src
rm -rf dist
unzip src.zip
npm run build'''
      }
    }

  }
}