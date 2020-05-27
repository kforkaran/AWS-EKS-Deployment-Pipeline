pipeline {
	agent any
	stages {
		stage('Create kubernetes cluster') {
			steps {
				withAWS(region:'ap-south-1', credentials:'aws-static') {
					sh '''
						var=`aws eks describe-cluster --name deploymentCluster`
						if [[ $var == *"(ResourceNotFoundException)"*]]; then
							eksctl create cluster \
							--name deploymentCluster \
							--version 1.13 \
							--nodegroup-name standard-workers \
							--node-type t2.small \
							--nodes 2 \
							--nodes-min 1 \
							--nodes-max 3 \
							--node-ami auto \
							--region ap-south-1 \
							--zones ap-south-1a \
							--zones ap-south-1b \
							--zones ap-south-1c
						fi
					'''
				}
			}
		}
		stage('Create conf file cluster') {
			steps {
				withAWS(region:'ap-south-1', credentials:'aws-static') {
					sh '''
						aws eks --region ap-south-1 update-kubeconfig --name deploymentCluster
					'''
				}
			}
		}
		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh 'docker build -t karan1999/deploymentPipeline .'
				}
			}
		}
		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push karan1999/deploymentPipeline
					'''
				}
			}
		}
		stage('Set current kubectl context') {
			steps {
				withAWS(region:'ap-south-1', credentials:'aws-static') {
					sh 'kubectl config use-context arn:aws:eks:ap-south-1:389695700925:cluster/deploymentCluster'
				}
			}
		}
		stage('Deploy blue container') {
			steps {
				withAWS(region:'ap-south-1', credentials:'aws-static') {
					sh 'kubectl apply -f ./blue-controller.json'
				}
			}
		}
		stage('Deploy green container') {
			steps {
				withAWS(region:'ap-south-1', credentials:'aws-static') {
					sh 'kubectl apply -f ./green-controller.json'
				}
			}
		}
		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'ap-south-1', credentials:'aws-static') {
					sh 'kubectl apply -f ./blue-service.json'
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
				withAWS(region:'ap-south-1', credentials:'aws-static') {
					sh 'kubectl apply -f ./green-service.json'
				}
			}
		}
	}
}