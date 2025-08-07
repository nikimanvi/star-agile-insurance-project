pipeline {
    agent { label 'maven_slave' } 

    environment {
        DOCKER_IMAGE = "nikithamanvi/insurance:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/nikimanvi/star-agile-health-care.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker Image: ${DOCKER_IMAGE}"
                sh """
                    docker build -t ${DOCKER_IMAGE} .
                    docker tag ${DOCKER_IMAGE} nikithamanvi/insurance:latest
                    docker image ls
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                echo 'Logging into DockerHub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerloginid',
                    usernameVariable: 'DOCKERHUB_CREDENTIALS_USR',
                    passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                        
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                }
            }
        }

        stage('Publish the Image to DockerHub') {
            steps {
                echo "Publishing Docker Image: ${DOCKER_IMAGE}"
                sh """
                    docker push ${DOCKER_IMAGE}
                    docker push nikithamanvi/insurance:latest
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'eks-master',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'deployment.yaml,service.yaml',
                                        removePrefix: '',
                                        remoteDirectory: '',
                                        remoteDirectorySDF: false,
                                        flatten: true,
                                        execCommand: '''
                                            mkdir -p /home/devopsadmin/deploy &&
                                            mv deployment.yaml service.yaml /home/devopsadmin/deploy/ &&
                                            kubectl apply -f /home/devopsadmin/deploy/deployment.yaml &&
                                            kubectl apply -f /home/devopsadmin/deploy/service.yaml
                                        ''',
                                        execTimeout: 120000
                                    )
                                ],
                                usePromotionTimestamp: false,
                                verbose: true
                            )
                        ]
                    )
                }
            }
        }
    }
}
