pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        // Environment variables
        DB_CONTAINER_NAME = "car_rental_db"
        BACKEND_CONTAINER_NAME = "car_rental_backend"
        FRONTEND_CONTAINER_NAME = "car_rental_frontend"
        // Force CI to false to ignore CRA warnings
        CI = 'false'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git...'
                git branch: 'main', url: 'https://github.com/OgunYildirim/Car-Rental-Full-Stack-Project.git' // Replace with your repo
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building Backend...'
                    dir('car-rental-system-backend') {
                        // Assuming running on Windows agent or container with Maven
                         bat 'mvn clean package -DskipTests'
                    }
                    
                    echo 'Building Frontend...'
                    dir('car-rental-system-frontend') {
                         bat 'npm install'
                         bat 'npm run build'
                    }
                }
            }
        }

        stage('Unit Tests') {
            steps {
                dir('car-rental-system-backend') {
                    echo 'Running Backend Unit Tests...'
                    bat 'mvn test'
                }
            }
            post {
                always {
                    junit 'car-rental-system-backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Integration Tests') {
             steps {
                dir('car-rental-system-backend') {
                    echo 'Running Backend Integration Tests...'
                    // Typically 'mvn verify' runs integration tests (failsafe plugin)
                    // Ensuring we run the UserResourceTest we created
                     bat 'mvn verify -DskipUnitTests' 
                }
            }
        }

        stage('Docker Deploy') {
            steps {
                echo 'Deploying to Docker...'
                // Ensure docker-compose is installed
                // Force remove any existing containers with these names (even if not part of this compose project)
                // Using "|| exit 0" to suppress error if containers don't exist
                bat 'docker rm -f car_rental_db car_rental_backend car_rental_frontend || exit 0'
                bat 'docker-compose down --volumes'
                bat 'docker-compose up -d --build'
                
                // Wait for services to be ready
                sleep time: 60, unit: 'SECONDS'
            }
        }

        stage('E2E: UI Smoke Tests') {
            steps {
                dir('automation-tests') {
                    echo 'Running Common UI Tests...'
                    bat 'mvn clean test -Dtest=CommonTests'
                }
            }
             post {
                always {
                     junit 'automation-tests/target/surefire-reports/*.xml'
                }
            }
        }

        stage('E2E: Auth Boundary Tests') {
            steps {
                dir('automation-tests') {
                    echo 'Running Auth Edge Cases...'
                    bat 'mvn clean test -Dtest=AuthMoreTest'
                }
            }
             post {
                always {
                     junit 'automation-tests/target/surefire-reports/*.xml'
                }
            }
        }

        stage('E2E: Full Auth Flows') {
            steps {
                dir('automation-tests') {
                    echo 'Running Critical Auth Flows...'
                    bat 'mvn clean test -Dtest=AuthTest'
                }
            }
            post {
                always {
                     junit 'automation-tests/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Expose to Internet with Ngrok') {
            steps {
                echo 'Starting Ngrok on port 3000...'
                // Eski ngrok işlemlerini kapat
                bat 'taskkill /F /IM ngrok.exe || exit 0'
                // Ngrok'u arka planda başlat (3000 portu frontend için docker-compose'da map edilmiş)
                bat 'start /b ngrok http 3000 > ngrok.log'
                // Başlaması için kısa bir süre bekle
                sleep time: 10, unit: 'SECONDS'
                // ngrok API'sinden atanan geçici public URL'i al ekrana yazdır
                script {
                    def ngrokUrl = powershell(script: "(Invoke-RestMethod -Uri 'http://localhost:4040/api/tunnels').tunnels[0].public_url", returnStdout: true).trim()
                    echo "Uygulamanız geçici olarak internete açıldı. Adres: ${ngrokUrl}"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline Finished.'
        }
        failure {
            echo 'Pipeline Failed!'
        }
    }
}
