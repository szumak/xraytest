#!/usr/bin/env groovy

def server = Artifactory.server "mszArtifactory"
pipeline {

    environment {
        ARTIFACTORY_REPO = credentials('ARTIFACTORY_HOST')
        ARTIFACTORY_CRED = credentials('ARTIFACTORY_DOCKER')
        IMAGE_NAME       = "${JOB_NAME}".replaceAll('/', '_').toLowerCase()
    }
   
    agent { node { label 'slave' } }

    stages {
        stage('Prepare') {
          steps {
             script {
                TAG = "${env.CHANGE_ID ? 'PR-' + env.CHANGE_ID : env.GIT_BRANCH}-${BUILD_ID}-${env.GIT_COMMIT.substring(0, 7)}".replaceAll('/', '_')
             }
          }
        }

        stage('Build image') {
            steps {
                echo 'Building...'
                sh """
                   cd image
                   docker build --pull -t "${ARTIFACTORY_REPO}/acme/${IMAGE_NAME}:${TAG}" .
                """
            }
        }
        stage('Artifactory push image and scan') {
            steps {
              script {
                def rtDocker = Artifactory.docker server: server
                image = "$ARTIFACTORY_REPO/acme/$IMAGE_NAME:$TAG"
                def buildInfo = rtDocker.push(image, 'acme')
                buildInfo.env.capture = true
                // Publish the merged build-info to Artifactory
                server.publishBuildInfo buildInfo
              }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
