pipeline {
    agent {
        label 'JAVAAPP'
    }
    stages {
        stage('git checkout'){
            steps{
                git url: 'https://github.com/raja23412/spring-petclinic.git',
                branch: 'main'
            }
        }
        stage('build and scan'){
            steps{
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
            stage('upload jfrog') {
                steps {
                    rtupload (
                        serverId: 'JFROG_ID_JAVA',
                        spec: '''{
                              "files": [
                                {
                                "pattern": "target/*.jar",
                                "target": "jfrogjavaspc-libs-release/"
                                }
                            ]
                        }''',
                    )
                    rtPublishBuildInfo (
                        serverId: 'JFROG_ID_JAVA',
                    )
                }
            }
              post {
        always { 
            archiveArtifacts artifacts: '**/target/*.jar'
            junit '**/target/surefire-reports/*.xml'
          } 
        success {
            echo 'this pipeline good'
        }   
        failure {
           echo 'this is waste pipeline'
        }      
    }

        }
    }
}