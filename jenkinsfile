pipeline {
	agent any
	stages {
		stage('Load ENV Vars') {
			steps {
				load 'prod-vars.groovy'
			}
		}
		stage('Deploy') {
			steps {
				script {
					openshift.withCluster( "${CLUSTER}" ) {
						openshift.withProject( "${ENVIRONMENT}" ) {
							openshift.create( openshift.process( "--filename", "example.yaml", "-p", "ENV_VAR_1=${ENV_VAR_1}", "-p", "ENV_VAR_2=${ENV_VAR_2}", "-p", "ENVIRONMENT=${ENVIRONMENT}" ))
						}
					}
				}
			}
		}
	}
}
