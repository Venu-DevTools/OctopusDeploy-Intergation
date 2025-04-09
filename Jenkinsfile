pipeline {
    agent any

    tools {
        maven 'Maven3' // Ensure Maven3 is configured in Jenkins Global Tool Configuration
    } 

    environment {
        OCTOPUS_API_KEY = credentials('octopus-api-key') // Securely pulls Octopus API key from Jenkins credentials
        OCTOPUS_PROJECT = 'helloworld'                   // Name of the Octopus Deploy project
        OCTOPUS_SPACE = 'firefist'                       // Name of the space in Octopus Deploy
        OCTOPUS_ENVIRONMENT = 'development'              // Target environment to deploy to
        OCTOPUS_SERVER = 'https://devtools.octopus.app/' // Your Octopus instance URL
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"          // Dynamic version: 1.0.1, 1.0.2, etc.
    }

    triggers {
        githubPush() // Automatically trigger the pipeline on push to GitHub
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
                // CHANGE: Use dot (.) instead of dash (-) to name the JAR properly for Octopus
                // Octopus parses "hello-world.1.0.8.jar" as:
                //   Package ID: hello-world
                //   Version: 1.0.8
                sh "mvn clean package -Djar.finalName=hello-world.${PACKAGE_VERSION}"
            }
        }

        stage('Archive JAR') {
            steps {
                // Archive the correctly named JAR for record-keeping in Jenkins
                archiveArtifacts artifacts: "target/hello-world.${PACKAGE_VERSION}.jar", fingerprint: true
            }
        }

        stage('Push to Octopus') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
                    //  CHANGE: Explicitly provide --packageId and --version to avoid parsing issues
                    sh '''
                        octo push \
                        --package target/hello-world.${PACKAGE_VERSION}.jar \
                        --server ${OCTOPUS_SERVER} \
                        --apiKey ${OCTO_API_KEY} \
                        --space "${OCTOPUS_SPACE}" \
                     '''
                }
            }
        }

        stage('Create Release') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
                    //  CHANGE: Explicitly link the correct package and version to the release
                    sh '''
                        octo create-release \
                        --project "${OCTOPUS_PROJECT}" \
                        --version ${PACKAGE_VERSION} \
                        --server ${OCTOPUS_SERVER} \
                        --apiKey ${OCTO_API_KEY} \
                        --space "${OCTOPUS_SPACE}" \
                        --package hello-world:${PACKAGE_VERSION}
                    '''
                }
            }
        }

        stage('Deploy Release') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
                    // Trigger the actual deployment to your chosen environment
                    sh '''
                        octo deploy-release \
                        --project "${OCTOPUS_PROJECT}" \
                        --version ${PACKAGE_VERSION} \
                        --server ${OCTOPUS_SERVER} \
                        --apiKey ${OCTO_API_KEY} \
                        --space "${OCTOPUS_SPACE}" \
                        --deployTo "${OCTOPUS_ENVIRONMENT}" \
                        --progress
                    '''
                }
            }
        }
    }
}
