pipeline {
   
   agent any
	
   options {
      //clean old builds
      buildDiscarder(logRotator(artifactDaysToKeepStr: '5', daysToKeepStr: '5', numToKeepStr: '6', artifactNumToKeepStr: '6'))
      //colorise output
      ansiColor('xterm')
   }
   
   tools {
      // Install the Maven version configured as "maven" and add it to the path.
      maven "maven"
   }
   
   stages {
         //get new code from repository
        stage('CHECKOUT') {
            steps {
                git 'https://github.com/allainmoyo/spring-boot.git'
            }
        }
	   
         //building the code to get new artifact
        stage('BUILD') {
            steps {
                sh "mvn clean install -f ./spring-boot-tests/spring-boot-smoke-tests/spring-boot-smoke-test-web-ui/pom.xml"
				sh "echo ${BUILD_NUMBER} > ~/build.number" //save build number to file to use it in QA/CI deployment jobs
            }
        }
	   
         //upload artifact to the Nexus3 repository
        stage("UPLOAD ARTIFACT") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'localhost:8081',
                    groupId: 'java',
                    version: 'build-${BUILD_NUMBER}',
                    repository: 'maven-repository',
                    credentialsId: 'nexus-credentials',
                    artifacts: [
                        [artifactId: 'spring-boot-smoke-test-web-ui',
                         classifier: '',
                         file: 'spring-boot-tests/spring-boot-smoke-tests/spring-boot-smoke-test-web-ui/target/spring-boot-smoke-test-web-ui-2.2.1.BUILD-SNAPSHOT.jar',
                         type: 'jar']
                    ]
                )
            }
        }
	   
         // deployment to QA instance and CI instance
        stage ('DEPLOY') {
            steps {
                build job: 'deploy_jar_QA'
                build job: 'deploy_jar_CI'
            }
        }
    }
}
