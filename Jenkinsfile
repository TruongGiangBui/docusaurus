pipeline {
    agent any
    environment {
        IMAGE_NAME = 'docusaurus'  
        SWARM_SERVICE = 'docusaurus'        
        MANAGER_IP = '103.48.84.175'     
        GIT_SERVICE_CREDS = "github-truonggiangbui"
        DOCKER_HUB_CREDENTIALS = "docker-truonggiangbui"
        CONTEXT_PATH = "${env.WORKSPACE}/source"  // Sử dụng thư mục chứa Dockerfile
    }
    stages {
        stage("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Git Clone") {            
            steps {
                dir("${env.WORKSPACE}/source") {
                    checkout([$class: 'GitSCM', branches: [[name: "master"]], extensions: [], userRemoteConfigs: [[credentialsId: GIT_SERVICE_CREDS, url: "https://github.com/TruongGiangBui/docusaurus.git"]]])
                    script {
                        lastCommit = sh(returnStdout: true, script: 'git rev-parse --verify HEAD').trim()
                        echo "=============== Last commit: ${lastCommit}"
                    }
                }
            }
        }
        stage('Build Image') {
            steps {
                script {
                    try {
                        echo "Starting to build Docker image..."
                        withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            dockerImage = docker.build("truonggiangbui/docusaurus", "--no-cache --network=host -f ${CONTEXT_PATH}/Dockerfile ${CONTEXT_PATH}")
                        }
                        echo "======== Build image successfully ========"
                    } catch (Exception e) {
                        error "======== Build image failed: ${e.message} ========"
                    }
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    try {
                        echo 'Starting to push Docker image...'
                        withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS) {
                                dockerImage.push("${env.BUILD_NUMBER}")
                                dockerImage.push("latest")
                            }
                        }
                        echo "======== Push image successfully ========"
                    } catch (Exception e) {
                        error "======== Push image failed: ${e.message} ========"
                    }
                }
            }
            post {
                success {
                    echo "======== Removing local Docker images ========"
                    sh 'docker rmi truonggiangbui/docusaurus:latest || true'
                    sh 'docker rmi truonggiangbui/docusaurus:${env.BUILD_NUMBER} || true'
                }
            }
        }
    }
}
