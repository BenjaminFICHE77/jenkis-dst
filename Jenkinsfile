pipeline {
    agent any
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
        // stage ('Testing') {
            // steps {
                // sh 'python -m unittest'
            // }
        // }
        stage ('Deploying') {
            steps {
                script {
                    sh '''
                        docker rm -f jenkins
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                        docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        // stage ('User acceptance') {
        //     steps {
        //         input {
        //             message "Voulez vous déployez sur la branche main ?"
        //             ok "Yes"
        //         }   
        //     }
        // }
        stage('Pushing and Merging') {
            parallel {
                stage('Pushing Image') {
                    // environment {
                    //     DOCKERHUB_CREDENTIALS = credentials('docker_jenkins')
                    // }
                    steps {
                        sh 'echo 7FiBen7!120 | docker login -u benjaminfiche77 --password-stdin'
                        sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                    }
                }
                stage ('Merging') {
                    steps {
                        echo 'Merging done'
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}