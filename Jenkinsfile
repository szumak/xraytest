pipeline {

    environment {
        ARTIFACTORY_REPO = credentials('ARTIFACTORY_HOST')
        ARTIFACTORY_CRED = credentials('ARTIFACTORY_DOCKER')
        IMAGE_NAME       = "${JOB_NAME}".replaceAll('/', '_').toLowerCase()
    }
   
    agent { node { label 'slave' } }

    stages {
        stage('Build image') {
            steps {
                echo 'Building...'
                sh """
                   cd image
                   docker build --pull -t "${ARTIFACTORY_REPO}/acme/${IMAGE_NAME}:latest" .
                """
            }
        }
        stage('Artifactory push image and scan') {
            steps {
              script {
                def rtDocker = Artifactory.docker server: server
                image = "$ARTIFACTORY_REPO/acme/$IMAGE_NAME:latest"
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
