pipeline {
    agent any

    tools {
        maven 'Maven3' // Match with Jenkins tool config
    }

    environment {
        OCTOPUS_API_KEY = credentials('octopus-api-key') // From Jenkins credentials
        OCTOPUS_PROJECT = 'helloworld'
        OCTOPUS_SPACE = 'firefist'
        OCTOPUS_ENVIRONMENT = 'development' // Change if you want to deploy to another environment
        OCTOPUS_SERVER = 'https://devtools.octopus.app/'
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}" // Version tied to the build number
    }

    triggers {
        githubPush()
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
                sh "mvn clean package -Djar.finalName=hello-world-${PACKAGE_VERSION}"
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: "target/hello-world-${PACKAGE_VERSION}.jar", fingerprint: true
            }
        }

        stage('Push to Octopus') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
                    sh '''
                        octo push \
                        --package target/hello-world-${PACKAGE_VERSION}.jar \
                        --server ${OCTOPUS_SERVER} \
                        --apiKey ${OCTO_API_KEY} \
                        --space "${OCTOPUS_SPACE}"
                    '''
                }
            }
        }

        stage('Create Release') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
                    sh '''
                        octo create-release \
                        --project "${OCTOPUS_PROJECT}" \
                        --version ${PACKAGE_VERSION} \
                        --server ${OCTOPUS_SERVER} \
                        --apiKey ${OCTO_API_KEY} \
                        --space "${OCTOPUS_SPACE}" \
                        --packageVersion ${PACKAGE_VERSION}
                    '''
                }
            }
        }

        stage('Deploy Release') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
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
