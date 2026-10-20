pipeline {
    agent none
    environment {
        IMAGE = 'ankush1223/webapp'
        TAG   = "${env.BRANCH_NAME.replace('/', '-')}-${BUILD_NUMBER}"
    }
    stages {
        stage('build') {
            agent { label 'built-in' }
            steps {
                checkout scm
                sh 'docker build -t $IMAGE:$TAG .'
            }
        }
        stage('test') {
            agent { label 'built-in' }
            steps {
                sh """
                    docker rm -f test-web || true
                    docker run -d --name test-web -p 8081:80 $IMAGE:$TAG
                    for i in 1 2 3 4 5 6; do
                        curl -fs http://localhost:8081 > /dev/null && exit 0
                        sleep 5
                    done
                    docker ps -a
                    docker logs test-web
                    exit 1
                """
            }
            post {
                always {
                    sh 'docker rm -f test-web || true'
                }
            }
        }
        stage('prod') {
            when { branch 'master' }
            stages {
                stage('push image') {
                    agent { label 'built-in' }
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                            sh '''
                                echo $DH_PASS | docker login -u $DH_USER --password-stdin
                                docker push $IMAGE:$TAG
                            '''
                        }
                    }
                }
                stage('deploy') {
                    agent { label 'prod' }
                    steps {
                        sh '''
                            docker pull $IMAGE:$TAG
                            docker rm -f webapp || true
                            docker run -d --name webapp -p 80:80 $IMAGE:$TAG
                        '''
                    }
                }
            }
        }
    }
}