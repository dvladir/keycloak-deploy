pipeline {
    options {
        disableConcurrentBuilds()
    }

    agent any

    environment {
        BRANCH="${BRANCH_NAME.replaceAll('feat/', '').toLowerCase()}"
        FOLDER="keycloak-${BRANCH}"
    }

    stages {
        stage('Show environment') {
            steps {
                sh 'echo BRANCH $BRANCH'
                sh 'echo FOLDER $FOLDER'
            }
        }
        stage('Prepare configs') {
            steps {
                configFileProvider([
                    configFile(fileId: "keycloak_conf_$BRANCH", targetLocation: './keycloak-deploy/.env')
                ]) {}
            }
        }
        stage('Build themes') {
            agent {
                docker {
                    image 'docker.dvladir.work/library/maven:3.8.6-openjdk-18'
                    args '--net=host'
                    reuseNode true
                }
            }
            steps {
                dir('./keycloak-theme') {
                    sh 'mvn clean package'
                    sh 'cp ./target/*.jar ../keycloak-deploy/deployments/'
                }
            }
        }
        stage('Deploy') {
            environment {
                DEPLOY_HOST = credentials('deploy-host')
                DEPLOY_PASS = credentials('deploy-pass')
                DEPLOY_PORT = "8787"
            }
            steps {
                sh 'echo ${DEPLOY_PASS} >> pass'
                sh 'sshpass -Ppassphrase -f ./pass rsync -rv ./keycloak-deploy/ ${DEPLOY_HOST}:~/${FOLDER}'
                sh 'sshpass -Ppassphrase -f ./pass ssh ${DEPLOY_HOST} cd \\~/${FOLDER} \\&\\& DEPLOY_PORT=${DEPLOY_PORT} BRANCH=${BRANCH} docker stack deploy --compose-file docker-compose.yml ${FOLDER}'
                sh 'rm ./pass'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}