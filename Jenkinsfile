


node {
    withEnv(['VERSION=latest',
             'PROJECT=todo',
             'CLIENT_IMAGE=client',
             'SERVER_IMAGE=server',
             'DB_IMAGE=mongo',
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
                    def VERSION =gitCommitHash.take(7)

                    sh('docker stop $(docker ps -aq)')

                    sh  """   
                        export VERSION=$VERSION 
                        export CLIENT_TAG=$VERSION
                        export BACKEND_TAG=$VERSION
                        export DB_TAG=$VERSION
                        printenv
                        docker -v
                        docker-compose -v
                        docker-compose build --no-cache
                        docker images
                    """
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

                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    def VERSION =gitCommitHash.take(7)


                    docker.withRegistry("$ECRURL", "$ECRCRED")   
                    {
                        docker.image("$CLIENT_IMAGE:$VERSION").push("$VERSION")
                        docker.image("$SERVER_IMAGE:$VERSION").push("$VERSION")
                        docker.image("$DB_IMAGE:latest").push("latest")
                    }

            
                    sh """
                    
                        /usr/local/bin/aws cloudformation update-stack --stack-name task --use-previous-template --parameters ParameterKey=VERSION,ParameterValue=$VERSION
                        /usr/local/bin/aws ecs update-service --force-new-deployment --service serivce-Service-1RFVUQ1RNF4EL
                    """
                    
                } catch(exc) {
                    throw exc
                }
            }      
        }        
    } 
}