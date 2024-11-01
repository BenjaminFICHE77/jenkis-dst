pipeline {
    environment { // Declaration of environment variables
    DOCKER_ID = "benjaminfiche77" // replace this with your docker-id
    DOCKER_IMAGE = "jenkins-exam"
    DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
    }
    agent any // Jenkins will be able to select all available agents
    stages {
        stage('Docker Build'){ // docker build image stag
            steps {
                script {
                sh '''
                 docker rm -f cast
                 docker rm -f movie
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG-cast cast-service/.
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG-movie movie-service/.
                 docker tag $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG-cast $DOCKER_ID/$DOCKER_IMAGE:latest-cast
                 docker tag $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG-movie $DOCKER_ID/$DOCKER_IMAGE:latest-movie
                sleep 6
                '''
                }
            }
        }

        stage('Docker run'){ // run container from our builded image
            steps {
                script {
                sh '''
                docker run -d -p 8001:8000 --name cast $DOCKER_ID/$DOCKER_IMAGE:latest-cast
                docker run -d -p 8002:8000 --name movie $DOCKER_ID/$DOCKER_IMAGE:latest-movie
                sleep 10
                '''
                }
            }
        }

        // stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
        //     steps {
        //         script {
        //         sh '''
        //         curl localhost:8001
        //         curl localhost:8002
        //         '''
        //         }
        //     }
        // }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {
                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE:latest-cast
                docker push $DOCKER_ID/$DOCKER_IMAGE:latest-movie
                '''
                }
            }
        }

        stage('Deploiement en dev'){
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
                    // cp fastapi/values.yaml values.yml
                    // cat values.yml
                    // sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    // helm upgrade --install app fastapi --values=values.yml --namespace dev
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install helm-chart-dev ./Helm --set namespace=dev
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
                helm upgrade --install helm-chart-staging ./Helm --set namespace=staging
                '''
                }
            }
        }

        stage('Deploiement en qa'){
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
                helm upgrade --install helm-chart-qa ./Helm --set namespace=qa
                '''
                }
            }
        }

    stage('Deploiement en prod'){
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install helm-chart-prod ./Helm --set namespace=prod
                '''
                }
            }
        }
    }
}