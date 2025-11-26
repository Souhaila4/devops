pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
    }
    
    environment {
        MAVEN_HOME = "${tool 'M2_HOME'}"
        PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"
        DOCKER_IMAGE = "souhaila4/student-management"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Souhaila4/devops'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        echo "📄 Création du Dockerfile..."
                        cat > Dockerfile << EOF
FROM eclipse-temurin:17-jre-alpine
COPY target/student-management-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8089
ENTRYPOINT ["java", "-jar", "app.jar"]
EOF

                        echo "✅ Dockerfile créé avec succès"
                        echo "🐳 Image prête pour le build manuel"
                    """
                }
            }
        }

        stage('Generate Docker Commands') {
            steps {
                script {
                    // Générer un script de déploiement
                    sh """
                        cat > deploy-docker.sh << EOF
#!/bin/bash
echo "🐳 Building Docker image..."
docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} .
docker tag ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ${env.DOCKER_IMAGE}:latest

echo "🔐 Login to DockerHub..."
docker login -u souhaila4

echo "🚀 Pushing to DockerHub..."
docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
docker push ${env.DOCKER_IMAGE}:latest

echo "✅ Done! Image: ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
EOF

                        chmod +x deploy-docker.sh
                        echo "📜 Script de déploiement créé: deploy-docker.sh"
                    """
                }
            }
        }
    }

    post {
        success {
            echo """
            🎉 BUILD RÉUSSI ! 🎉

            Étapes manuelles restantes :

            1. 📋 Copiez ces commandes ou exécutez le script :

            docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} .
            docker tag ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ${env.DOCKER_IMAGE}:latest
            docker login -u souhaila4
            docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
            docker push ${env.DOCKER_IMAGE}:latest

            2. 🔗 Vérifiez sur DockerHub :
            https://hub.docker.com/r/souhaila4/student-management

            3. 🐳 Pour tester l'image :
            docker run -p 8089:8089 ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
            """

            // Sauvegarder les commandes dans un artifact
            sh """
                echo 'docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} .' > docker-commands.txt
                echo 'docker tag ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ${env.DOCKER_IMAGE}:latest' >> docker-commands.txt
                echo 'docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}' >> docker-commands.txt
                echo 'docker push ${env.DOCKER_IMAGE}:latest' >> docker-commands.txt
            """
            archiveArtifacts artifacts: 'docker-commands.txt,deploy-docker.sh', fingerprint: true
        }
        failure {
            echo '❌ Pipeline a échoué!'
        }
    }
}
