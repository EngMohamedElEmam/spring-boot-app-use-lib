@Library('dockerize@main') _

// Set your parameters
def DOCKER_REGISTRY = 'docker.io/mohamedemam2020'
def DOCKER_IMAGE = 'spring-boot-app'
def credentialsId = "docker-creds"

pipeline {
    agent any

    environment {
        OPENSHIFT_PROJECT = 'nti'
        OPENSHIFT_SERVER = 'https://api.ocpuat.devopsconsulting.org:6443'
        DOCKER_REGISTRY = 'docker.io/mohamedemam2020'
        DOCKER_IMAGE = 'spring-boot-app'
        APP_SERVICE_NAME = 'spring-boot-app'
        APP_PORT = '8080'
        APP_HOST_NAME = 'spring-boot-app.apps.ocpuat.devopsconsulting.org'
    }
    
    // Define a variable to store the commit hash
    def COMMIT_HASH

    stages {
        stage('Get Commit Hash') {
            steps {
                script {
                    // Get the short commit hash and store it in COMMIT_HASH
                    COMMIT_HASH = dockerize.getCommitHash()
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/EngMohamedElEmam/spring-boot-app-use-lib'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerize.buildDockerImage(DOCKER_REGISTRY, DOCKER_IMAGE, COMMIT_HASH)
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    dockerize.pushDockerImage(DOCKER_REGISTRY, DOCKER_IMAGE, COMMIT_HASH, credentialsId)
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'openshift-credentials', variable: 'OPENSHIFT_SECRET')]) {
                    sh "oc login --token=\${OPENSHIFT_SECRET} \${OPENSHIFT_SERVER} --insecure-skip-tls-verify"
                    }
                    sh "oc project \${OPENSHIFT_PROJECT}"
                    sh "oc delete dc,svc,deploy,ingress,route \${DOCKER_IMAGE} || true"
                    sh "oc new-app \${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${COMMIT_HASH}"
                    sh "oc create route edge --service \${APP_SERVICE_NAME} --port \${APP_PORT} --hostname \${APP_HOST_NAME} --insecure-policy Redirect"
                }
            }
        }
    }
}