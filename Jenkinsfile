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
            sh '''
              set -eu
              rm -rf dist
              mkdir -p dist
              cp index.html dist/index.html
              {
                echo "git_commit=${GIT_COMMIT:-unknown}"
                echo "build_number=${BUILD_NUMBER:-unknown}"
                echo "build_url=${BUILD_URL:-unknown}"
                date -u +"built_at=%Y-%m-%dT%H:%M:%SZ"
              } > dist/build-info.txt
            '''
          } else {
            bat '''
              if exist dist rmdir /s /q dist
              mkdir dist
              copy index.html dist\\index.html
              (
                echo git_commit=%GIT_COMMIT%
                echo build_number=%BUILD_NUMBER%
                echo build_url=%BUILD_URL%
                powershell -NoProfile -Command "'built_at=' + [DateTime]::UtcNow.ToString('yyyy-MM-ddTHH:mm:ssZ')"
              ) > dist\\build-info.txt
            '''
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
              docker exec "$NGINX_CONTAINER" sh -c 'rm -rf /usr/share/nginx/html/*'
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
              docker exec "%NGINX_CONTAINER%" sh -c "rm -rf /usr/share/nginx/html/*"
              docker cp dist/. "%NGINX_CONTAINER%:/usr/share/nginx/html/"
            '''
          }
        }
        echo "Deployed to nginx at http://localhost:${env.NGINX_PORT}/"
      }
    }

    stage('Verify Deploy') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              set -eu
              source_hash="$(sha256sum dist/index.html | awk '{print $1}')"
              target_hash="$(docker exec "$NGINX_CONTAINER" sha256sum /usr/share/nginx/html/index.html | awk '{print $1}')"
              echo "workspace index.html sha256=$source_hash"
              echo "nginx index.html sha256=$target_hash"
              test "$source_hash" = "$target_hash"
              docker exec "$NGINX_CONTAINER" test -s /usr/share/nginx/html/build-info.txt
            '''
          } else {
            bat '''
              powershell -NoProfile -Command "$source=(Get-FileHash -Algorithm SHA256 'dist/index.html').Hash.ToLowerInvariant(); $target=(docker exec $env:NGINX_CONTAINER sha256sum /usr/share/nginx/html/index.html).Split()[0].ToLowerInvariant(); Write-Host ('workspace index.html sha256=' + $source); Write-Host ('nginx index.html sha256=' + $target); if ($source -ne $target) { throw 'nginx index.html was not updated' }"
              docker exec "%NGINX_CONTAINER%" test -s /usr/share/nginx/html/build-info.txt
            '''
          }
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'dist/**', fingerprint: true
    }
  }
}
