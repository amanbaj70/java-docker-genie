pipeline {
    agent any
    
    tools{
        jdk 'jdk-17'
        maven 'maven3.8'
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
        // PROJECT_NAME = '${JOB_NAME}'
        scanner_key = 'squ_f3b4978f34ce862df6b527d41f6382427fd80e64'
        CHECK_URL = "http://34.100.251.114:8081/DockerProducts/"
        CMD = "curl --write-out %{http_code} --silent --output /dev/null ${CHECK_URL}"
    }
    stages {
        stage('Git checkout') {
            steps {
                
                git branch: 'main', url: 'https://github.com/amanbaj70/java-docker-genie.git'
                sh " ls -l ${WORKSPACE}"
            }
        }
        
         stage('Maven Code compile') {
            steps {
                sh "mvn compile"    
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                sh '''
                    ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.url=http://34.100.251.114:9000/ -Dsonar.login=${scanner_key} \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=${JOB_NAME}
                '''   
            }
        }
        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher  pattern: '**/dependency-check-report.xml' 
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Docker Build and Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred'){
                    sh 'docker build -t docker-desktop .'
                    sh 'docker tag docker-desktop amanbajpai/docker-desktop:v1'
                    sh 'docker push amanbajpai/docker-desktop:v1'
                    }
                }
            }
        }
        
        stage('Docker Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred'){
                        sh 'docker rm -f docker-desktop'
                        sh 'docker run --name docker-desktop -d -p 8085:8080 amanbajpai/docker-desktop:v1'
                    }
                }
            }
        }

        // stage('Stage-One') {
        //     steps {
        //         script{
        //             sh "${CMD} > commandResult"
        //             sh "cat commonResult"
        //             // env.status = readFile('commandResult').trim()
        //         }
        //     }
        // }
    }
}