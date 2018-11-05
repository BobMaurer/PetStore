pipeline {
    agent any
	environment{
		DOCKERNAME="petstore_${env.BRANCH_NAME}:${env.BUILD_NUMBER}"
		DBNAME="petstore_${env.BRANCH_NAME}"
		DBHOST="localhost"
		KUBEDBHOST="petstore-mysql.default.svc.cluster.local"
		KUBECONFIG="c:\\Users\\hturowski\\.kube\\config"
		NAMESPACE="${env.SERVICENAME}-${env.BRANCH_NAME}"
		SERVICENAME="petstore"
        SERVICE_PORT=80
	}

    stages {
       stage('Database Migration') {
            steps {
                echo 'Applying database migrations..'
				dir("PetStore") {
                	bat "dotnet ef database update"
				}
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Unit testing..'
                bat "dotnet test ./PetStore.Tests"
            }
        }

        stage('Build Docker Container') {
            steps {
                echo 'Building..'
				bat "docker build -t ${env.DOCKERNAME} ."
            }
        }

        // stage('Configure Namespace') {
        //     steps {
		// 		bat "kubectl --kubeconfig ${env.KUBECONFIG} create namespace ${env.NAMESPACE}"
		// 	}
        // }

        stage('Deploy') {
            steps {
				bat "kubectl --kubeconfig ${env.KUBECONFIG} set env deployments/${env.SERVICENAME} DBNAME=${env.DBNAME} -n ${env.NAMESPACE}"
				bat "kubectl --kubeconfig ${env.KUBECONFIG} set env deployments/${env.SERVICENAME} DBHOST=${env.KUBEDBHOST} -n ${env.NAMESPACE}"
				bat "kubectl --kubeconfig ${env.KUBECONFIG} set image deployments/${env.SERVICENAME} ${env.SERVICENAME}=${env.DOCKERNAME} -n ${env.NAMESPACE}"
            }
        }

        stage('Integration Test') {
            steps {
                echo 'Integration Testing..'
                if(env.BRANCH_NAME != "master") {
                   SERVICE_PORT=81 
                }
                bat "dotnet test ./PetStore.Integration.Tests"
            }
			post {
				failure {
					echo 'Rolling back..'
					bat "kubectl rollout undo deployments/${env.SERVICENAME} -n ${env.NAMESPACE}"
				}
			}
        }
    }
}
