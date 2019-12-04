#!groovy
pipeline {
    agent any
    options {
    buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    tools {
    maven 'M2_HOME'
    }
    stages {
        stage('Build') {
            steps {
		// Run the maven build
                echo 'Clean Build'
		    
                   // echo 'Pulling.......' + env.BRANCH_NAME
                
          }
            }
        
        stage('Test') {
            steps {
		    // Run integration test
                echo 'Testing'
                sh 'mvn test'
            }
        }
	stage('JaCoCo') {
            steps {
                echo 'Code Coverage'
                jacoco()
            }
        }
        stage('sonar'){
	    steps {
		     // Run the sonar scan
               echo 'Sonar Scanner'
		   // sh "mvn sonar:sonar -Dsonar.host.url=http://198.198.10.46:9000 -Dsonar.login=XXXXXXXXX"
		    withSonarQubeEnv('SonarQube') {

                        //sh "'${mvnHome}/bin/mvn'  verify sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true"
                   sh " mvn verify sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true"
	    }}
	 }
		// waiting for sonar results based into the configured web hook in Sonar server which push the status back to jenkins
	    stage('Sonar scan result check') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    retry(3) {
                        script {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                        }
                    }
                }
            }

}
}
}
post {
    success {
      notifyBuild()
        //notifyBuild('SUCCESSFUL')
    }
    failure {
      notifyBuild('ERROR')
    }
  }
    }
// Slack notification with status and code changes from git
def notifyBuild(String buildStatus = 'SUCCESSFUL') {
  //buildStatus = buildStatus
    buildStatus =  buildStatus ?: 'SUCCESSFUL'

  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job Name - '${env.JOB_NAME}', Build No. - '${env.BUILD_NUMBER}', Git Commit - '${env.GIT_COMMIT}'"
  def changeSet = getChangeSet()
  def message = "${subject} \n ${changeSet}"

  if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  slackSend (color: colorCode, message: message)
}

@NonCPS

// Fetching change set from Git
def getChangeSet() {
  return currentBuild.changeSets.collect { cs ->
    cs.collect { entry ->
        "* ${entry.author.fullName}: ${entry.msg}"
    }.join("\n")
  }.join("\n")
}
