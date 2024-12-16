pipeline {
    agent none
    environment {
        BUILD_SERVER = 'ec2-user@10.0.0.75'
        IMAGE_NAME = 'sandeep888/repo1:php$BUILD_NUMBER'
        DOCKER_SERVER = 'ec2-user@10.0.0.223'
    }
    stages {
        
        stage('Bulid the docker image and push to dockerhub') {
            agent any
            steps {
                script {
                    sshagent(['Build']) {
                        withCredentials([usernamePassword(credentialsId: 'dockerlogin', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            echo "Building the docker image of PHP application"
                            sh "scp -o StrictHostKeyChecking=no -r docker-files ${BUILD_SERVER}:/home/ec2-user"
                            sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/docker-files/docker-script.sh'"
                            sh "ssh ${BUILD_SERVER} sudo docker build -t ${IMAGE_NAME} /home/ec2-user/docker-files"
                            sh "ssh ${BUILD_SERVER} sudo docker login -u $USERNAME -p $PASSWORD"
                            sh "ssh ${BUILD_SERVER} sudo docker push ${IMAGE_NAME}"
                        }
                    }
                
                }
                
            }

            
        }
        stage('Deploying Docker container using Docker Compose') {
            agent any
            steps {
                script {
                    sshagent(['Docker']) {
                        withCredentials([usernamePassword(credentialsId: 'dockerlogin', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            sh "scp -o StrictHostKeyChecking=no -r docker-files ${DOCKER_SERVER}:/home/ec2-user"
                            sh "ssh -o StrictHostKeyChecking=no ${DOCKER_SERVER} 'bash ~/docker-files/docker-script.sh'"
                            sh "ssh ${DOCKER_SERVER} sudo docker login -u $USERNAME -p $PASSWORD"
                            sh "ssh ${DOCKER_SERVER} bash /home/ec2-user/docker-files/docker-compose-script.sh ${IMAGE_NAME}"
                        }
                    }
                }
            }
        }               
    }
}
