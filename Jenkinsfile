// at the pipeline and stage level
pipeline {
    agent any
    environment {
         nom = 'datascientest'
    }

    environment {
        DOCKER_ID = "benjaminfiche77"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {
        stage ('Building') {
            steps {
                sh 'pip install -r requirements.txt'
            }

        }

        stage ('Testing') {
            steps {
                sh 'python -m unittest'
            }

        }

        stage ('Deploying') {
            steps {
                sh '''
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAB .
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
            }

        }

        stage ('User acceptance') {
            setps {
                input {
                    message "Voulez vous déployez sur la branche main ?"
                    ok "Yes"
                }   
            }
        }

        parallel {
            stage('Pushing Image') {
                environment {
                    DOCKERHUB_CREDENTIALS = credentials('docker_jenkins')
                }
                steps {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
            stage () {
                echo 'Merging done'
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}