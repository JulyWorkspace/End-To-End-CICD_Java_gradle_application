pipeline{
    agent any
    environment{
        VERSION= "${env.BUILD_ID}"
    }
    stages{
        stage("Checking version"){
            steps{
                sh 'java --version'
                sh 'mvn -version'
            }
        }
        stage('Code Qaulity'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonartoken') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }

        } 
        stage('Docker build and Docker push'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'docker-nexus-passwd', variable: 'passwd')]) {
                    sh '''
                    docker build -t 3.6.43.239:8083/springapp:${VERSION} .
                    docker login -u admin -p $passwd 3.6.43.239:8083
                    docker push 3.6.43.239:8083/springapp:${VERSION}
                    docker rmi 3.6.43.239:8083/springapp:${VERSION}
                    '''
		   }
                }
            }
        }
        // stage('Indentifying misconfigs using datree in helm charts'){
        //     steps{
        //         script{
        //             dir('kubernetes/') {
        //                 withEnv(['DATREE_TOKEN=27aeb149-5cc2-4f61-a362-1e27fe99bd14']) {
        //                     sh 'helm datree test myapp/'
        //                 }
                        
        //             }
        //         }
        //     }

        // }
         stage('Pushing the Helm Chatrs to Nexus'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'docker-nexus-passwd', variable: 'passwd')]) {
                    dir('kubernetes/') {
                        sh '''
                        helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                        tar -czvf myapp-${helmversion}.tgz myapp/
                        curl -u admin:$passwd http://3.6.43.239:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                        '''
                        }
                    }
                }
            }
        }
	    stage('Deploying application on k8s cluster') {
	            steps {
	               script{
			       withCredentials([string(credentialsId: 'K8s', variable: 'passwd')]) {
				       dir('kubernetes/') {
					       sh 'helm upgrade --install --set image.repository="3.6.43.239:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
				       }
	                    	}
	               	}
	            }
	        }
    }

    }
	
     post {
         always {
			mail bcc: '', body:"<br>Project: ${env.JOB_NAME} <br> Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL} <br> Status: ${currentBuild.result}</b>", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "kunalumrane@gmail.com";  
		 }
}

// pipeline{
//     agent any 
//     environment{
//         VERSION = "${env.BUILD_ID}"
//     }
//     stages{
//         stage("sonar quality check"){
//             agent {
//                 docker {
//                     image 'openjdk:11'
//                 }
//             }
//             steps{
//                 script{
//                     withSonarQubeEnv(credentialsId: 'sonar-token') {
//                             sh 'chmod +x gradlew'
//                             sh './gradlew sonarqube'
//                     }

//                     timeout(time: 1, unit: 'HOURS') {
//                       def qg = waitForQualityGate()
//                       if (qg.status != 'OK') {
//                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
//                       }
//                     }

//                 }  
//             }
//         }
//         stage("docker build & docker push"){
//             steps{
//                 script{
//                     withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
//                              sh '''
//                                 docker build -t 34.125.214.226:8083/springapp:${VERSION} .
//                                 docker login -u admin -p $docker_password 34.125.214.226:8083 
//                                 docker push  34.125.214.226:8083/springapp:${VERSION}
//                                 docker rmi 34.125.214.226:8083/springapp:${VERSION}
//                             '''
//                     }
//                 }
//             }
//         }
//         stage('indentifying misconfigs using datree in helm charts'){
//             steps{
//                 script{

//                     dir('kubernetes/') {
//                         withEnv(['DATREE_TOKEN=GJdx2cP2TCDyUY3EhQKgTc']) {
//                               sh 'helm datree test myapp/'
//                         }
//                     }
//                 }
//             }
//         }
//         stage("pushing the helm charts to nexus"){
//             steps{
//                 script{
//                     withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
//                           dir('kubernetes/') {
//                              sh '''
//                                  helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
//                                  tar -czvf  myapp-${helmversion}.tgz myapp/
//                                  curl -u admin:$docker_password http://34.125.214.226:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
//                             '''
//                           }
//                     }
//                 }
//             }
//         }

//         stage('manual approval'){
//             steps{
//                 script{
//                     timeout(10) {
//                         mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@gmail.com";  
//                         input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
//                     }
//                 }
//             }
//         }

//         stage('Deploying application on k8s cluster') {
//             steps {
//                script{
//                    withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
//                         dir('kubernetes/') {
//                           sh 'helm upgrade --install --set image.repository="34.125.214.226:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
//                         }
//                     }
//                }
//             }
//         }

//         stage('verifying app deployment'){
//             steps{
//                 script{
//                      withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
//                          sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'

//                      }
//                 }
//             }
//         }
//     }

//     post {
// 		always {
// 			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@gmail.com";  
// 		 }
// 	   }
// }
