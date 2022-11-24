pipeline{

    agent any

    environment {

        VERSION = "${env.BUILD_ID}"
    }

    stages{

        stage('sonar quality check'){

            agent{

                docker {
                    image 'maven'
                }
            }
            steps{

                script{

                    withSonarQubeEnv(credentialsId: 'sonar-token'){

                        sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate status'){

            steps{

                script{

                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }

        stage('docker build & docker push to nexus repo'){

            steps{

                script{
                   withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_creds')]) {

                    sh '''
                     docker build -t nexus_ip:8083/springapp:${VERSION} .

                     docker login -u admin -p $nexus_creds nexus_ip:8083

                     docker push nexus_ip:8083/springapp:${VERSION}

                     docker rmi nexus_ip:8083/springapp:${VERSION}
                    '''
                    }

                    
                }
            }
        }
        stage('Identifying misconfigs using datree in helm charts') {

            steps{
                script{
                    dir('kubernetes/myapp/'){
                         withEnv(['DATREE_TOKEN= ']) {
                        sh 'helm datree test .'
                        }
                    }
                }
            }
        }

        stage('Push to helm chart in nexus repo') {

            steps {
                script{
                    withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_creds')]) {
                        dir('kubernetes/myapp/'){

                    sh '''
                    helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d '' )
                    tar -czvf myapp-${helmversion}.tgz myapp/
                    curl -u admin:$nexus_creds http://nexus_machine_ip:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                    '''
                }
                }
            }
            }
        }
    }

    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "vikash.mrdevops@gmail.com";  
		}
	}
}