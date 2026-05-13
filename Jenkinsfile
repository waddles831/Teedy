pipeline {
    agent any
    stages {
        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }
        stage('Install') {   
            steps {
                sh 'mvn install -DskipTests -Dpmd.skip=true -Dcpd.skip=true -Dcheckstyle.skip=true'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test -Dmaven.test.failure.ignore=true'
            }
        }
        stage('PMD') {
            steps {
                sh 'mvn pmd:pmd'
            }
        }
        stage('JaCoCo') {
            steps {
                sh 'mvn jacoco:report'
            }
        }
        stage('Javadoc') {
            steps {
                sh 'mvn javadoc:javadoc'
            }
        }
        stage('Site') {
            steps {
                sh 'mvn site'
            }
        }
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t waddles831/teedy .'
            }
        }

        stage('Docker Push') {
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