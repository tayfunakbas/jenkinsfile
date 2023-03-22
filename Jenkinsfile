pipeline {
    agent any
    stages {
        
        stage('Get Source Code From Github') {
            steps{
           sh '''docker logout
           echo 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 | docker login -u tayfunakbas ghcr.oi/tayfunakbas/codestock/nginx:0.0.1
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
