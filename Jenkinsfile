


node {
    withEnv(['VERSION=latest',
             'PROJECT=todo',
             'CLIENT_IMAGE=client',
             'SERVER_IMAGE=server',
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
                    def VERSION =gitCommitHash.take(7)

                    sh "export VERSION=$VERSION"
                    sh "export CLIENT_TAG=$VERSION"
                    sh "export SERVER_TAG=$VERSION"

                    sh  '''    
                        docker -v
                        docker stop $(docker ps -aq)
                        docker-compose -v
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
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    def VERSION =gitCommitHash.take(7)
                    docker.withRegistry("$ECRURL", "$ECRCRED")   
                    {
                        docker.image("$CLIENT_IMAGE").push(sh("echo $VERSION"))
                        docker.image("$SERVER_IMAGE").push("$VERSION")
                        docker.image("$DB_IMAGE").push("$VERSION")
                    }
                    // sh '''
                    //        aws ecs update-service --cluster "$CLUSTER" --service "$SERVICE" --task-definition "$TASK_DEFINITION":"$REVISION"
                    //       /usr/local/bin/aws cloudformation update-stack --template-body file://$PWD/infra/create-task-definition.yml --stack-name task
                    // '''
                    
                } catch(exc) {
                    throw exc
                }
            }      
        }        
    } 
}