
pipeline {
    agent {
         label 'JAVAAPP'
          }
           triggers {
       pollSCM('* * * * *')
    }
    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/raja23412/spring-petclinic.git'
            }
        }

        stage('Build and Sonar Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar_id', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SONARQUBE') {
                        sh '''
                            mvn clean verify sonar:sonar \
                              -Dsonar.projectKey=raja23412_spring-petclinic \
                              -Dsonar.organization=rajajenkins \
                              -Dsonar.host.url=https://sonarcloud.io \
                              -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload to JFrog') {
            steps {
                rtUpload (
                    serverId: 'JFROG_ID_JAVA',
                    spec: '''{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "jfrogjavaspc-libs-release-local/"
                            }
                        ]
                    }'''
                )
                rtPublishBuildInfo (
                    serverId: 'JFROG_ID_JAVA'
                )
            }
        }
    }

    post {
        always { 
            archiveArtifacts artifacts: '**/target/*.jar'
            junit '**/target/surefire-reports/*.xml'
        } 
        success {
            echo 'Pipeline executed successfully!'
        }   
        failure {
            echo 'Pipeline failed. Please check logs.'
        }      
    }
}
