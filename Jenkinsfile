pipeline {
    agent any
    options { 
        timestamps() // Affiche les horodatages dans les logs
    }
    
    environment {
        IMAGE = 'zayneeb/monapp'
        TAG = "build-${env.BUILD_NUMBER}" // Chaque build a un tag unique
    }

    stages {
        // -----------------------------
        stage('Checkout') {
            steps {
                echo '=== Étape 0 : Récupération du code source ==='
                checkout scm // Lit le même repo que le job Jenkins
            }
        }

        // -----------------------------
        stage('Docker Build') {
            steps {
                echo '=== Étape 1 : Construction de l’image Docker ==='
                bat 'docker version' // Vérifie que Docker est installé
                bat "docker build -t %IMAGE%:%TAG% ." // Construit l’image Docker
            }
        }

        // -----------------------------
        stage('Smoke Test') {
            steps {
                echo '=== Étape 2 : Smoke Test de l’image ==='
                bat """
                docker rm -f monapp_test 2>nul || ver > nul
                docker run -d --name monapp_test -p 8081:80 %IMAGE%:%TAG%
                ping -n 3 127.0.0.1 > nul
                curl -I http://localhost:8081 | find "200 OK"
                docker rm -f monapp_test
                """
            }
        }

        // -----------------------------
        stage('Push (Docker Hub)') {
            steps {
                echo '=== Étape 3 : Push de l’image sur Docker Hub ==='
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                usernameVariable: 'USER',
                                                passwordVariable: 'PASS')]) {
                    bat """
                    echo %PASS% | docker login -u %USER% --password-stdin
                    docker tag %IMAGE%:%TAG% %IMAGE%:latest
                    docker push %IMAGE%:%TAG%
                    docker push %IMAGE%:latest
                    """
                }
            }
        }
    }

    // -----------------------------
    post {
        success { echo 'Build + Test + Push OK ✅' }
        failure { echo 'Build / Test / Push KO ❌' }
    }
}
