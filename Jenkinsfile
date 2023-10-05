pipeline {
    agent any
    
    environment {
        S3_BUCKET = 'sathis.backup'
        S3_PATH = 'backup'
        DEFAULT_RECIPIENTS = 'ssselvasathis@gmail.com'
        DEFAULT_REPLYTO = 'selvasathis28@gmail.com'
        DEFAULT_SUBJECT = 'backup process state'
        
        CURRENT_DATE = sh(script: 'date +%Y%m%d_%H%M%S', returnStdout: true).trim()
    }

    stages {
        stage('Fetch Docker Containers') {
            steps {
                script {
                    DOCKER_CONTAINERS = sh(script: "docker ps --format '{{.Names}}'", returnStdout: true).trim()
                    echo "Fetched Docker Containers: ${DOCKER_CONTAINERS}"
                }
            }
        }

        stage('Convert Containers to Images') {
            steps {
                script {
                    def containersArray = DOCKER_CONTAINERS.split('\n')
            
                    containersArray.each { container ->
                        sh "docker commit ${container} ${container}:${env.CURRENT_DATE}"
                        echo "Image created for ${container}."
                    }
                }
            }
        }

        stage('Save Images and Backup to S3') {
            steps {
                script {
                    DOCKER_IMAGES = sh(script: 'docker image ls --format "{{.Repository}}:{{.Tag}}"', returnStdout: true).trim()
                    def imagesArray = DOCKER_IMAGES.split('\n')

                    imagesArray.each { image ->
                        def isImageUsed = sh(script: "docker ps -q --filter ancestor=${image}", returnStdout: true).trim()
                        if (!isImageUsed) {
                            sh "docker save -o ${image}.tar ${image}"
                            sh "aws s3 cp ${image}.tar s3://${S3_BUCKET}/${S3_PATH}/${image}.tar"
                            sh "rm ${image}.tar"
                            sh "docker rmi ${image}"
                        } else {
                            echo "Skipping removal of ${image} as it is in use by a running container."
                        }
                    }
                }
            }
        }
    }
}
