pipeline {
    agent any
    stages {
        
        stage('Get Source Code From Github') {
           echo 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 | docker login -u tayfunakbas ghcr.io
           docker pull ghcr.io/tayfunakbas/codestock/nginx:0.0.1
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
            docker tag ghcr.io/tayfunakbas/codestock/nginx:0.0.1 ghcr.io/tayfunakbas/codestock/nginx:0.0.2
        }
        stage('Deploy to Docker'){
            docker run -it --port 8080:40000 -d ghcr.io/tayfunakbas/codestock/nginx:0.0.2
        }        
    }
}
