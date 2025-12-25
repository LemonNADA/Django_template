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
                sh 'docker run -d --name django-container -p 9007:8000 my-django-app'
            }
        }
        stage('Nginx') {
            steps {
                sh 'ssh -o -i ${KEY_FILE} ${USERNAME}@${SERVER_ADDRESS} sudo certbot --nginx --non-interactive --agree-tos -m paramonov@informatics.ru -d ${DOMAIN_NAME}'
                sh 'ssh -i ${KEY_FILE} ${USERNAME}@${SERVER_ADDRESS} sudo systemctl reload nginx'
            }
        }
    }
}