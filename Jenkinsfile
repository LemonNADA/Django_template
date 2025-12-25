pipeline {
    agent any
    stages {
        stage('Build Image') {
            steps {
                sh 'docker build -t my-django-app .'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker stop django-container'
                sh 'docker rm django-container'
                sh 'docker run -d --name django-container -p 9007:8000 my-django-app'
            }
        }
        stage('Nginx') {
            steps {
                sh 'scp -i ${KEY_FILE} paramonov.prod.mshp-devops.conf ${USERNAME}@${PROD_IP}:nginx'
                sh 'ssh -i ${KEY_FILE} ${USERNAME}@${PROD_IP} sudo systemctl reload nginx'
            }
        }
    }
}