pipeline {
      agent any

      environment {
          APACHE_LOG = '/var/log/apache2/access.log'
          ERROR_LOG  = '/var/log/apache2/error.log'
      }

      stages {
          stage('Install Apache2') {
              steps {
                  sh '''
                      sudo apt-get update -y
                      sudo apt-get install -y apache2
                      sudo systemctl enable apache2
                      sudo systemctl restart apache2
                  '''
              }
          }

          stage('Verify Apache is running') {
              steps {
                  sh '''
                      systemctl is-active apache2
                      curl -I http://localhost/
                  '''
              }
          }

          stage('Generate fake errors') {
              steps {
                  sh '''
                      curl -s -o /dev/null -w "GET /missing  -> %{http_code}\\n" http://localhost/this-page-does-not-exist
                      curl -s -o /dev/null -w "GET /missing2 -> %{http_code}\\n" http://localhost/another-missing-404

                      sudo a2enmod cgi || true
                      printf '#!/bin/bash\\nexit 1\\n' | sudo tee /usr/lib/cgi-bin/broken.cgi >/dev/null
                      sudo chmod +x /usr/lib/cgi-bin/broken.cgi
                      sudo systemctl reload apache2
                      curl -s -o /dev/null -w "GET /cgi-bin/broken.cgi -> %{http_code}\\n" http://localhost/cgi-bin/broken.cgi || true
                  '''
              }
          }

          stage('Check logs for 4xx/5xx') {
              steps {
                  sh '''
                      echo "=== 4xx/5xx entries in access.log ==="
                      sudo awk '$9 ~ /^[45][0-9][0-9]$/ {print}' "$APACHE_LOG" || true

                      echo "=== Counts ==="
                      echo -n "4xx: "; sudo awk '$9 ~ /^4[0-9][0-9]$/' "$APACHE_LOG" | wc -l
                      echo -n "5xx: "; sudo awk '$9 ~ /^5[0-9][0-9]$/' "$APACHE_LOG" | wc -l
                  '''
              }
          }
      }

      post {
          always { echo 'Pipeline finished.' }
      }
  }
