 pipeline {
    agent any

    environment {
        // Variables pour Nexus
        registryCredentials = "nexus" // Le nom de tes credentials dans Jenkins
        registry = "localhost:8083"    // L'URL de ton Nexus
        imageName = "aymenjallouli_4twin3_parkini" // Ton nom d'image Docker
        imageTag = "latest-${env.BUILD_NUMBER}"  // Version dynamique de l'image Docker

        // Variables pour SonarQube
        SONAR_URL = "http://192.168.1.10:9000"    // URL de ton serveur SonarQube
        SONAR_TOKEN = "squ_ade158460ca7ba334c038b7f5ff8408f3e74196c" // Ton token SonarQube
        SONAR_PROJECT_KEY = "AymenJallouli_Parkini"
        SONAR_PROJECT_NAME = "AymenJallouli-Parkini"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Aymenjallouli/Parkini_test.git'  // URL de ton dépôt Git
            }
        }

       /* stage('Build & Test') {
            steps {
                // Exécution des tests et du build. Utilise un script adapté pour ton projet.
                sh 'npm install'  // Installer les dépendances Node.js
                sh 'npm run test'  // Lancer les tests
            }
        }*/

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Analyser le projet avec SonarQube
                    def scannerHome = tool 'scanner'  // Utilise le sonar-scanner installé sur Jenkins
                    withSonarQubeEnv('scanner') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                            -Dsonar.sources=src \
                            -Dsonar.language=js \
                            -Dsonar.javascript.lcov.reportPaths=coverage/lcov-report/index.html \
                            -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                // Construction de l'image Docker de ton projet
                sh "DOCKER_BUILDKIT=1 docker build -t $registry/$imageName:$imageTag ."
            }
        }

        stage('Push to Nexus') {
            steps {
                script {
                    // Push de l'image Docker vers Nexus
                    docker.withRegistry("http://$registry", registryCredentials) {
                        sh "docker push --quiet $registry/$imageName:$imageTag"
                    }
                }
            }
        }

        stage('Run Application') {
            steps {
                script {
                    // Exécution du déploiement avec docker-compose, utilisation de la dernière image
                    docker.withRegistry("http://$registry", registryCredentials) {
                        sh "docker pull $registry/$imageName:$imageTag"

                        // Remplacer dynamiquement l'image dans le fichier docker-compose.yml
                        sh "sed -i 's|IMAGE_TAG|$imageTag|g' docker-compose.yml"
                        sh "cat docker-compose.yml"  // Vérification du fichier

                        // Lancer les containers avec la nouvelle image
                        sh "docker-compose up -d --no-deps"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline exécuté avec succès !"
        }
        failure {
            echo "Le pipeline a échoué !"
        }
    }
}
