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

        stage('Unit Tests') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'cd car-rental-system-backend && mvn test'
                    } else {
                        bat 'cd car-rental-system-backend && mvn test'
                    }
                }
            }
            post {
                always {
                    junit 'car-rental-system-backend/target/surefire-reports/*.xml'
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
