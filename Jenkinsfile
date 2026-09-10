pipeline { 
	agent any 
	environment { 
		gcpCreds = 'gcp_credentials'
		dockerCreds = credentials('dockerhub_login') 
		registry = "${dockerCreds_USR}/vatcal" 
		registryCredentials = "dockerhub_login" 
		dockerImage = "" // empty var, will be written to later
		TF_VAR_gcp_project = "qadevoprac3-lab10-tfstate-1017-2003"
		TF_VAR_docker_registry = "${registry}"
	} 
	stages { 
		stage('Run Tests') { 
			steps { 
				sh 'npm install' 
				sh 'CI=true npm test' 
				} 
			} 
			stage('Build Image') { 
				steps { 
					script { 
						dockerImage = docker.build(registry) 
					} 
				}
		} 
		stage('Push Image') { 
			steps { 
				script { 
					docker.withRegistry("", registryCredentials) { 
						dockerImage.push("${env.BUILD_NUMBER}") 
						dockerImage.push("latest") 
					}
				}
			} 
		} 
		stage('Clean Up') { 
			steps { 
				sh "docker image prune --all --force --filter 'until=48h'" 
			} 
		} 
		stage('Deploy') { 
			steps { 
				script { 
					withCredentials([file(credentialsId: gcpCreds, variable: 'GCP_CREDENTIALS')]) { 
						sh '''
						export GOOGLE_APPLICATION_CREDENTIALS=$GCP_CREDENTIALS
						terrascan scan -i terraform -t gcp -p .  --non-recursive
						terraform init
						terraform plan -out plan.tfplan
						terraform apply plan.tfplan 
						'''
					}
				}
			}
		}
	} 
}
