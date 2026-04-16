pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DB_CONTAINER_NAME = "car_rental_db"
        BACKEND_CONTAINER_NAME = "car_rental_backend"
        FRONTEND_CONTAINER_NAME = "car_rental_frontend"
        CI = 'false'
    }

    stages {
        stage('Check Tools') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'which mvn || { echo "Maven yüklü değil!"; exit 1; }'
                        sh 'which npm || { echo "Node.js/npm yüklü değil!"; exit 1; }'
                        sh 'which docker || { echo "Docker yüklü değil!"; exit 1; }'
                        sh 'which docker-compose || { echo "docker-compose yüklü değil!"; exit 1; }'
                    } else {
                        bat 'where mvn || (echo Maven yüklü değil! & exit /b 1)'
                        bat 'where npm || (echo Node.js/npm yüklü değil! & exit /b 1)'
                        bat 'where docker || (echo Docker yüklü değil! & exit /b 1)'
                        bat 'where docker-compose || (echo docker-compose yüklü değil! & exit /b 1)'
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                echo 'Checking out code from Git...'
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'cd car-rental-system-backend && mvn clean package -DskipTests'
                    } else {
                        bat 'cd car-rental-system-backend && mvn clean package -DskipTests'
                    }
                }
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'cd car-rental-system-frontend && npm install && npm run build'
                    } else {
                        bat 'cd car-rental-system-frontend && npm install && npm run build'
                    }
                }
            }
        }




        stage('Docker Deploy') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker rm -f car_rental_db car_rental_backend car_rental_frontend || true'
                        sh 'docker-compose down --volumes'
                        sh 'docker-compose up -d --build'
                    } else {
                        bat 'docker rm -f car_rental_db car_rental_backend car_rental_frontend || exit 0'
                        bat 'docker-compose down --volumes'
                        bat 'docker-compose up -d --build'
                    }
                }
                sleep time: 60, unit: 'SECONDS'
            }
        }

        stage('Start ngrok Tunnels') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
# Çalışma dizinine ngrok indirilmemişse indir ve aç
if ! command -v ngrok >/dev/null 2>&1; then
  echo "ngrok bulunamadı; workspace içine indiriliyor..."
  if [ ! -f ngrok ]; then
    curl -s -L -o ngrok.zip https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
    unzip -o ngrok.zip || true
    chmod +x ngrok || true
  fi
fi
# Frontend ve backend için tünelleri başlat (frontend:3000, backend:8080)
./ngrok http 3000 --log=stdout > ngrok-frontend.log 2>&1 & echo $! > ngrok-frontend.pid || true
./ngrok http 8080 --log=stdout > ngrok-backend.log 2>&1 & echo $! > ngrok-backend.pid || true
sleep 6
# ngrok'un lokal API'sinden tünel bilgilerini çek
echo "--- ngrok tunnels (raw JSON) ---"
curl --silent --fail http://127.0.0.1:4040/api/tunnels || echo "ngrok API erişilemedi veya ngrok başlatılamadı"
                        '''
                    } else {
                        bat '''
where ngrok >nul 2>&1 || (
  echo ngrok bulunamadı, lütfen agent'a ngrok kurun. & exit /b 0
)
rem Frontend ve backend tünellerini arka planda başlat
start /B ngrok http 3000 > ngrok-frontend.log 2>&1
start /B ngrok http 8080 > ngrok-backend.log 2>&1
timeout /t 6 /nobreak >nul
powershell -NoProfile -Command "try { Invoke-RestMethod -Uri http://127.0.0.1:4040/api/tunnels | ConvertTo-Json -Depth 5 } catch { Write-Output 'ngrok API erişilemedi veya ngrok başlatılamadı' }"
                        '''
                    }
                }
            }
        }

    }

    post {
        always {
            echo 'Pipeline Finishedddddd.'
        }
        failure {
            echo 'Pipeline Failed!'
        }
    }
}
