pipeline {
    agent any

    options {
      // Don’t run more than one build at a time.
      disableConcurrentBuilds()
      // If it fails, retry one time.
      retry(1)
      // If a stage returns with an unstable status, skip the rest of the build.
      skipStagesAfterUnstable()
      // Clean old builds
      buildDiscarder(logRotator(artifactDaysToKeepStr: '5', daysToKeepStr: '5', numToKeepStr: '6', artifactNumToKeepStr: '6'))
      // Set the build is aborted if not concluded after 10 minutes.
      timeout(time: 10, unit: 'MINUTES')
      // Colorise output with plugin
      ansiColor('xterm')
   }
	
   environment {
	// Set Maven variablr to colorise output
        MAVEN_OPTS = '-Djansi.force=true'
    }
	
   tools {
      // Install the Maven version configured as "maven" and add it to the path.
      maven "maven"
    }
	
    stages {
	// Delete Dir
	stage('Delete Dir') {            
            steps {
                deleteDir()            
                }
        }
	
        // Get new code from repository
        stage('CHECKOUT') {
            steps {
		sh "echo ${JOB_NAME}"
		// sh "rm -rf spring-boot/"
                git 'https://github.com/allainmoyo/spring-boot.git'
            }
        }
	   
        // Build the code to get new artifact
        stage('BUILD') {
            steps {
                //maven builds (Run in non-interactive (batch) mode)
                sh "mvn -Dstyle.color=always -B clean install -f ./spring-boot-tests/spring-boot-smoke-tests/spring-boot-smoke-test-web-ui/pom.xml"
                //save build number to file to use it in QA/CI deployment jobs
                sh "echo ${BUILD_NUMBER} > ~/build.number" 
            }
        }
	
	   
        // Upload artifact to the Nexus3 repository
        stage('UPLOAD ARTIFACT') {
            steps {
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: 'localhost:8081',
                groupId: 'java',
                version: 'build-${BUILD_NUMBER}',
                repository: 'maven-repository',
                credentialsId: 'nexus-credentials',
                artifacts: [[
                    artifactId: 'spring-boot-smoke-test-web-ui',
                    classifier: '',
                    file: 'spring-boot-tests/spring-boot-smoke-tests/spring-boot-smoke-test-web-ui/target/spring-boot-smoke-test-web-ui-2.2.1.BUILD-SNAPSHOT.jar',
                    type: 'jar'
                    ]]
                )
            }
        }
	   
        // Deployment of artifact to QA and CI instances
        stage ('DEPLOY') {
            steps {
                build job: 'deploy_jar_QA'
                build job: 'deploy_jar_CI'
            }
        }
	      
        // Clean workspace
        stage('Clean workspace') {
          steps {
          cleanWs()
      }
    }
  }
}
