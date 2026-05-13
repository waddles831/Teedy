pipeline {
    agent any
    stages {
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
        stage('Building image') {
            steps {
                sh 'docker build -t waddles831/teedy .'
            }
        }

        stage('Upload Image') {
            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push waddles831/teedy
                    '''
                }
            }
        }

        stage('Run Containers') {
            steps {

                sh 'docker rm -f teedy1 teedy2 teedy3 || true'

                sh 'docker run -d --name teedy1 -p 8082:8080 waddles831/teedy'

                sh 'docker run -d --name teedy2 -p 8083:8080 waddles831/teedy'

                sh 'docker run -d --name teedy3 -p 8084:8080 waddles831/teedy'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/target/site/**/*.*', fingerprint: true
            archiveArtifacts artifacts: '**/target/**/*.jar', fingerprint: true
            archiveArtifacts artifacts: '**/target/**/*.war', fingerprint: true
            junit '**/target/surefire-reports/*.xml'
        }
    }
}