pipeline {
    agent any 

    tools {
        maven 'M2_HOME'
    }

    stages {

        stage('GIT') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Souhaila4/devops.git'            }
        }

        stage('Maven') {
            steps {
                sh 'mvn -version'
            }
        }

    }
}
