pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('dockerhub login') {
            steps {
                echo "============docker hub login============"
                withCredentials([usernamePassword(credentialsId: 'dockerhub_test', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "docker login -u $USERNAME -p $PASSWORD"
                }
            }
        }
        stage('Build docker image') {
            when {
                branch 'master'
            }
            steps {
                echo "============start building docker image============"
                sh 'docker build -t 948615654/train-testapp:${BUILD_NUMBER} .'
                echo "${env.BUILD_NUMBER}"
            }
        }
        stage('Push docker image') {
            when {
                branch 'master'
            }
            steps {
                echo "============start pushing docker image============"
                sh '''
                docker push 948615654/train-testapp:${BUILD_NUMBER}
                '''
            }
        }
        stage('Deploy image to production') {
            when {
                branch 'master'
            }
            steps {
                echo "============deploy to production============"
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'production_key', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull 948615654/train-testapp:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-testapp\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-testapp\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-testapp -p 8080:8080 -d 948615654/train-testapp:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
