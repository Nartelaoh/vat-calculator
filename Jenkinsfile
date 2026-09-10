pipeline { 
	agent any
	environment {
		dockerCreds - credentials('dockerhub_login')
		registry = "${dockerCreds_USR}/vatcal"
		registryCredentials = "dockerhub_login"
		dockerImage = "" // empty var, will be written to later
	}
	
	stages {
	// 	stage('Checkout'){ 
	// 		steps { 
	// 			git url: 'https://github.com/nartelaoh/vat-calculator.git', 
	// 			branch: 'main'
	// 			} 
		stage('Run Tests') { 
			steps { 
				sh 'npm install' 
				sh 'CI=true npm test'
			}
		}
		// stage('Archive') { 
		// 	steps { 
		// 		sh 'tar -czf build.tar.gz build' 
		// 		archiveArtifacts 'build.tar.gz' 
		// 	} 
		// } 
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
	} 
}
