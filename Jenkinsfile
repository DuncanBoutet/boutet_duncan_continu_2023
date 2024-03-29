pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "boudun" // replace this with your docker-id
DOCKER_IMAGE_MOVIE_SERVICE = "movie-service"
DOCKER_IMAGE_CAST_SERVICE = "cast-service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Print Git Info') {
            steps {
                script {
                    echo "GIT_BRANCH: ${env.GIT_BRANCH}"
                    echo "BRANCH_NAME: ${env.BRANCH_NAME}"
                }
            }
        }
		    
        stage('Docker Build movie-service'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG -f ./movie-service/Dockerfile ./movie-service
                sleep 6
                '''
                }
            }
        }
        stage('Docker run movie-service'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance movie-service'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push movie-service'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }
            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE_SERVICE:$DOCKER_TAG
                '''
                }
            }
        }
		
		stage('Docker Build cast-service'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG -f ./cast-service/Dockerfile ./cast-service
                sleep 6
                '''
                }
            }
        }
        stage('Docker run cast-service'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance cast-service'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push cast-service'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }
            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST_SERVICE:$DOCKER_TAG
                '''
                }
            }
        }

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp fastapi/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app fastapi --values=values.yml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en QA'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp fastapi/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app fastapi --values=values.yml --namespace qa
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp fastapi/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app fastapi --values=values.yml --namespace staging
                '''
                }
            }

        }
stage('Deploiement en prod') {
    environment {
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on Jenkins
    }
    steps {
        script {
            if (env.GIT_BRANCH == 'origin/master') {
                // Create an Approval Button with a timeout of 15 minutes.
                // This requires a manual validation in order to deploy on the production environment
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }

                // Perform deployment steps for production
                sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace prod
                '''
            } else {
                echo 'Skipping deployment to production as the branch is not master.'
            }
        }
    }
}

}
}
