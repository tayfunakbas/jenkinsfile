pipeline {
    agent any
    stages {
        
        stage('Get Source Code From Github') {
           echo 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 | docker login -u tayfunakbas ghcr.io/luciopanepinto/pacman.git
           docker pull ghcr.io:/tayfunakbas/codestock/nginx:0.0.1
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

        stage('Deploy to Docker'){
            docker run -it --port 8080:40000 -d ghcr.io/tayfunakbas/codestock/nginx:0.0.1
        }        
    }
}
