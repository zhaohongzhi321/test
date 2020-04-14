pipeline {
  agent any
  stages {
    stage('update') {
      steps {
        git(url: 'https://github.com/zhaohongzhi321/test.git', branch: 'master', changelog: true, credentialsId: 'f300575701a6d3473a9da0656a931eaa3b1953f8', poll: true)
      }
    }

    stage('build') {
      steps {
        sh '''pwd
cd ..
rm -rf ./base-project/src
cp -r ./test_master ./base-project/src
cd ./base-project/
rm -rf dist
npm run build
cd ./dist/build
zip -r h5.zip ./h5/
cd ../../../
cp ./base-project/dist/build/h5.zip ./test_master/'''
      }
    }

    stage('archive') {
      steps {
        archiveArtifacts(artifacts: '*.zip', allowEmptyArchive: true, caseSensitive: true, defaultExcludes: true, fingerprint: true)
      }
    }

  }
}