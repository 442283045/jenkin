pipeline {
  agent any

  environment {
    NGINX_CONTAINER = 'jenkins-test-nginx'
    NGINX_PORT = '8081'
  }

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
        script {
          if (isUnix()) {
            sh '''
              set -eu
              if ! docker ps --format '{{.Names}}' | grep -qx "$NGINX_CONTAINER"; then
                if docker ps -a --format '{{.Names}}' | grep -qx "$NGINX_CONTAINER"; then
                  docker rm -f "$NGINX_CONTAINER"
                fi
                docker run -d --name "$NGINX_CONTAINER" -p "$NGINX_PORT:80" nginx
              fi
              docker cp dist/. "$NGINX_CONTAINER:/usr/share/nginx/html/"
            '''
          } else {
            bat '''
              docker ps --format "{{.Names}}" | findstr /x "%NGINX_CONTAINER%" >nul
              if errorlevel 1 (
                docker ps -a --format "{{.Names}}" | findstr /x "%NGINX_CONTAINER%" >nul
                if not errorlevel 1 docker rm -f "%NGINX_CONTAINER%"
                docker run -d --name "%NGINX_CONTAINER%" -p "%NGINX_PORT%:80" nginx
              )
              docker cp dist/. "%NGINX_CONTAINER%:/usr/share/nginx/html/"
            '''
          }
        }
        echo "Deployed to nginx at http://localhost:${env.NGINX_PORT}/"
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'dist/**', fingerprint: true
    }
  }
}
