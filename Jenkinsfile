pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        script {
          if (isUnix()) {
            sh 'rm -rf dist && mkdir -p dist && cp index.html dist/index.html'
          } else {
            bat 'if exist dist rmdir /s /q dist && mkdir dist && copy index.html dist\\index.html'
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploy stage reached. Replace this with the real deployment command.'
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'dist/**', fingerprint: true
    }
  }
}
