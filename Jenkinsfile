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
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'Dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t johndev007/johncapstonecluster .
					'''
				}
			}
		}		

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'Dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD 
						docker push johndev007/johncapstonecluster
					'''
				}
			}
		}

		stage('Set the current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						kubectl config get-contexts
						kubectl config use-context arn:aws:eks:us-west-2:238894399712:cluster/JohnCapstoneCluster/
					'''
				}
			}
		}
		stage('Deploy blue the container') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
		
			}
		}
		
		stage('Deploy the green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to the blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

	stage('Wait for user to  approve') {
            steps {
                input "Are you ready to redirect traffic to green?"
            }
        }

		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-west-2', credentials:'Jenkins_User') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}
	}

}

