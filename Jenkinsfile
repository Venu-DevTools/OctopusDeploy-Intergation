pipeline {
    agent any

    tools {
        maven 'Maven3' // Match this name with what's configured in Jenkins -> Global Tool Configuration
    }

    environment {
        OCTOPUS_API_KEY = credentials('octopus-api-key') // Inject Octopus API Key from Jenkins credentials
    }

    triggers {
        githubPush() // Auto-trigger on GitHub push (make sure webhook is configured) 
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
                // This customizes the JAR name to match Octopus version
                sh "mvn clean package -Djar.finalName=hello-world-1.0.${BUILD_NUMBER}"
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: "target/hello-world-1.0.${BUILD_NUMBER}.jar", fingerprint: true
            }
        }

        stage('Push to Octopus') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
                    sh '''
                        octo push \
                        --package target/hello-world-1.0.${BUILD_NUMBER}.jar \
                        --server https://devtools.octopus.app/ \
                        --apiKey ${OCTO_API_KEY} \
                        --space "firefist"
                    '''
                }
            }
        }
    }
}
