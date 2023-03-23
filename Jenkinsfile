pipeline {
    agent any

    environment{
    GITHUB_USER_CREDENTIAL = credentials('githubcred')
    }
    
    stages {
        
        stage('Get Source Code From Github') {
            steps{
           sh '''docker logout
           docker login -u  ${GITHUB_USER_CREDENTIAL_PSW} -p ${GITHUB_USER_CREDENTIAL_USR}  ghcr.oi/tayfunakbas/codestock/nginx:0.0.1
           docker pull ghcr.oi/tayfunakbas/codestock/nginx:0.0.1'''
            }
        }

        stage('Build Stage'){
            steps {
            echo "Building..."
            }
        }

        stage('Push to Github'){
            steps {
            echo "Push to Github..."
            }
        }

        stage('Tag the Image'){
            steps {
            sh '''docker tag ghcr.io/tayfunakbas/codestock/nginx:0.0.1 nginx:0.0.2'''
            }
        }
        stage('Deploy to Docker'){
            steps{
            sh '''docker run -it --port 8080:40000 -d nginx:0.0.2'''
            }
        }        
    }
}
