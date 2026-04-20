pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ksheeru/app"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
	stage('Checkout') {
		steps {
			checkout scm
		}
	}
	stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                '''
            }
        }
	stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }
	stage('Push Image') {
            steps {
                sh '''
                docker push $DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }
	stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    cp deployment.yaml deployment-temp.yaml
                    export KUBECONFIG=$KUBECONFIG_FILE
                    sed -i "s|IMAGE_TAG|$BUILD_NUMBER|g" deployment.yaml

                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml

                    '''
                }
            }
        }
    }
