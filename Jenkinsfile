pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {
                checkout scm
            }
        }

        stage('Build Maven') {

            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {

            steps {
                sh 'sudo docker build -t rferns/webapp:latest .'
            }
        }

        stage('Docker Push') {

            steps {
		withCredentials([usernamePassword(
			credentialsId: 'docker-creds',
			usernameVariable: 'DOCKER_USER',
			passwordVariable: 'DOCKER_PASS'
		)]) {
			sh 'echo $DOCKER_PASS } sudo docker login -u $DOCKER_USER --password-stdin'
			sh 'sudo docker push rferns/webapp:latest'
		}
            }
        }

        stage('Kubernetes Deploy') {

            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}
