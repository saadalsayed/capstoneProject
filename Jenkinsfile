
pipeline {
	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'hubuser', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
					        docker build -t saad1996/capstoneimage .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'hubuser', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push saad1996/capstoneimage
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'jenkins') {
					sh '''
						kubectl config use-context arn:aws:iam::088990380082:role/eksctl-udacitycapstonecluster-cluster-ServiceRole-1MW1L4XYOEI0M
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'jenkins') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'jenkins') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-1', credentials:'jenkins') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-east-1', credentials:'jenkins') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}
