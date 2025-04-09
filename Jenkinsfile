pipeline {
    agent any

    tools {
        maven 'Maven3' // This must match the name in Global Tool Configuratio
    }

    triggers {
        githubPush() // Triggers build when a GitHub push event occurs (webhook)
    }

    stages {
        stage('Clone from GitHub') {
            steps {
                git credentialsId: 'github-creds', 
                    url: 'https://github.com/Venu-DevTools/OctopusDeploy-Intergation.git', 
                    branch: 'main'
            }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}
