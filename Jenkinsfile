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
npm run build
cd ./dist/build
zip -r h5.zip ./h5/
cd ../../../
cp ./base-project/dist/build/h5.zip ./test_master/'''
      }
    }

    stage('build2') {
      steps {
        archiveArtifacts(onlyIfSuccessful: true, artifacts: '*.zip', allowEmptyArchive: true, caseSensitive: true, defaultExcludes: true)
      }
    }

  }
}