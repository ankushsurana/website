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
                    sleep 5
                    curl -f http://localhost:8081
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