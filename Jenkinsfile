pipeline {
	agent any
	environment{
		registry='chance562/project2'
        	dockerImage=''
        	dockerHubCredentials='DockerHub'
	}
	stages{
		stage("Maven build"){
			steps{
				sh '/usr/local/apache-maven/bin/mvn clean package'
			}
		}
		stage("Docker build"){
			steps{
				script{
					dockerImage = docker.build "$registry"
				}
			}
		}
		stage("Pushing to DockerHub Registry"){
            		steps{
                		script{
                    			docker.withRegistry('',dockerHubCredentials){
                        			dockerImage.push("$currentBuild.number")
                        			dockerImage.push("latest")
                    			}
                		}
            		}
        	}
		stage("Waiting for approval"){
			steps{
				script{
					try {
						timeout(time: 6, unit: 'HOURS'){
							approved = input message: 'Deploy to production?', ok: 'Continue',
								parameters: [choice(name: 'approved', choices: 'Yes\nNo', description: 'Deploy this build to production')]
							if(approved != 'Yes'){
								error('Build not approved')
							}
						}
					} catch (error){
						error('Build not approved in time')
					}
				}
			}
		}
		stage("Deploying to Kubernetes production environment"){
			steps{
				script{
					sh 'kubectl delete -f yamlFiles_for_Deployment/bank-service-deployment.yml'
                                        sh 'kubectl apply -f yamlFiles_for_Deployment/bank-service-deployment.yml'
				}
			}
		}
	}
}
