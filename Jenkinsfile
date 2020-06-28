pipeline {
	agent any
	stages {

		stage('Linting') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo docker build -t johndev007/johncapstonecluster .
					'''
				}
			}
		}
		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD 
						sudo docker push johndev007/johncapstonecluster
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						sudo -s
						kubectl config get-contexts
						kubectl config use-context arn:aws:eks:us-west-2:238894399712:cluster/JohnCapstoneCluster
					'''
				}
			}
		}
		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
		
			}
		}
		
		stage('Deploy green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

	}

}

