


node {
    withEnv(['VERSION=latest',
             'PROJECT=todo',
             'CLIENT_IMAGE=client:latest',
             'SERVER_IMAGE=server:latest',
             'DB_IMAGE=mongo:latest',
             'ECRURL=http://308106623039.dkr.ecr.us-east-1.amazonaws.com',
             'ECRCRED=ecr:us-east-1:AWS_CRED' 
             ]) {

        stage('checkout scm'){
            git 'https://github.com/mohsin51/circle-ci-sample'
        }
        stage('build docker images'){
            script {
                try {
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    VERSION = gitCommitHash.take(7)
                    sh  '''
                        docker -v
                        docker stop $(docker ps -aq)
                        docker-compose -v
                        export CLIENT_TAG=$VERSION
                        export SERVER_TAG=$VERSION
                        docker-compose build --no-cache
                        docker images
                    '''
                } catch(exce) {
                    sh 'docker-compose down'
                    throw exce
                }
            }
        }
        stage('push images to ecr'){
            script
            {
                try {
                    AWS_ACCESS_KEY_ID=credentials('jenkins-aws-secret-key-id')
                    AWS_SECRET_ACCESS_KEY=credentials('jenkins-aws-secret-access-key')
                    sh  '''
                        /usr/local/bin/aws ecr get-login --no-include-email | sh
                    '''   
                    docker.withRegistry("$ECRURL", "$ECRCRED")
                    {
                        docker.image("$CLIENT_IMAGE").push()
                    }
                    docker.withRegistry("$ECRURL", "$ECRCRED")
                    {
                        docker.image("$SERVER_IMAGE").push()
                    }
                    docker.withRegistry("$ECRURL", "$ECRCRED")
                    {
                        docker.image("$DB_IMAGE").push()
                    }
                    
                } catch(exc) {
                    throw exc
                }
            }      
        }        
    } 
}