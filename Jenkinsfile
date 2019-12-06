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
		sh 'mvn clean compile'
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
        
        stage('Sonar-Static-Scanning'){
	        steps {
		        // Run the sonar scan
                echo 'Sonar Scanner'
		        // sh "mvn sonar:sonar -Dsonar.host.url=http://198.198.10.46:9000 -Dsonar.login=XXXXXXXXX"
		        withSonarQubeEnv('SonarQube') {
                    sh " mvn verify sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true"
	            }
            }
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
      
	    stage('Deploy-from-featurebranch') {
            when {
		        expression {
		        return env.GIT_BRANCH == "origin/feature"

                }   
            }
            steps {
                echo 'Deploying to QA Environment'
                    //sshagent(['dev-server']) {
                    //sh "rsync -ivhr $WORKSPACE/ServiceInterface/bin/ -e 'ssh -o StrictHostKeyChecking=no' '${env.devsfws}':'/usr/share/nginx/www/DevRubyWS/bin/'"
                    //sh "ssh -o StrictHostKeyChecking=no '${env.devsfws}' 'sudo chmod +x /usr/share/nginx/www/DevRubyWS/bin'"
                //}
		    }   
        }
	    
	    stage('Deploy-from-develop-branch') {
            when {
		        expression {
		        return env.GIT_BRANCH == "origin/develop"

                }   
            }
            steps {
                echo 'Deploying to Stag Environment'
                    //sshagent(['dev-server']) {
                    //sh "rsync -ivhr $WORKSPACE/ServiceInterface/bin/ -e 'ssh -o StrictHostKeyChecking=no' '${env.devsfws}':'/usr/share/nginx/www/DevRubyWS/bin/'"
                    //sh "ssh -o StrictHostKeyChecking=no '${env.devsfws}' 'sudo chmod +x /usr/share/nginx/www/DevRubyWS/bin'"
                //}
		    }   
        }
	    
	    stage('Deploy-from-master-branch') {
            when {
		        expression {
		        return env.GIT_BRANCH == "origin/master"

                }   
             }
            steps {
                echo 'Deploying to Prod Environment'
                    //sshagent(['dev-server']) {
                    //sh "rsync -ivhr $WORKSPACE/ServiceInterface/bin/ -e 'ssh -o StrictHostKeyChecking=no' '${env.devsfws}':'/usr/share/nginx/www/DevRubyWS/bin/'"
                    //sh "ssh -o StrictHostKeyChecking=no '${env.devsfws}' 'sudo chmod +x /usr/share/nginx/www/DevRubyWS/bin'"
                //}
		    }   
        }

        stage('Arachni-Dynamic-Scanning') {
	    steps {
		  echo 'Arachni-Dynamic-Scanning'
                  sh 'rm -rf /var/jenkins_home/workspace/arachni_report/*'
                  sh 'rm -rf /var/jenkins_home/workspace/pipeline-jenkinstest/arachni_report/*'
	          //sh '/arachni-1.4-0.5.10/bin/arachni http://35.171.80.62:8080 '
                  sh '/arachni-1.4-0.5.10/bin/arachni http://35.171.80.62:8080 --report-save-path=/var/jenkins_home/workspace/arachni_report/arachni_report.afr'
	          sh '/arachni-1.4-0.5.10/bin/arachni_reporter /var/jenkins_home/workspace/arachni_report/arachni_report.afr --reporter=html:outfile=/var/jenkins_home/workspace/arachni_report/arachni_report.html.zip'
                  sh 'cp /var/jenkins_home/workspace/arachni_report/arachni_report.html.zip /var/jenkins_home/workspace/pipeline-jenkinstest/arachni_report'
	          sh 'chmod 777 -R /var/jenkins_home/workspace/pipeline-jenkinstest/arachni_report'
                  //sh 'unzip /var/jenkins_home/workspace/pipeline-jenkinstest/arachni_report/arachni_report.html.zip -d arachni_report'
		    sh 'unzip $WORKSPACE/arachni_report/arachni_report.html.zip -d $WORKSPACE/arachni_report'
            }
      }
	stage('Publish HTML report') {
	    steps {
		  echo 'Arachni-Dynamic-Scanning Report'
	          publishHTML target: [
                  allowMissing: false,
                  alwaysLinkToLastBuild: false,
                  keepAll: true,
                  reportDir: 'arachni_report',
                  reportFiles: 'index.html',
                  reportName: 'Arachni-Dynamic-Scanning Report'
                  ]
	    }
	 }
	    
	stage('Blaze-Meter') {
            steps {
                echo 'Blaze-Meter'
                blazeMeterTest credentialsId: 'blazemeter-api-key', testId: '7498756.functionalGui', workspaceId: '400136'
	        //blazeMeterTest credentialsId: 'blazemeter-api-key', testId: '7500672.functionalGui', workspaceId: '400136'
	    }
	 }
    }

    post {
        success {
            notifyBuild()
            //notifyBuild('SUCCESSFUL')
	    //slackUploadFile credentialId: 'jenkins-test-devsecops-slack', filePath: '/var/jenkins_home/workspace/pipeline-jenkinstest/arachni_report.html.zip', initialComment: 'Arachni Scanning Report'
            //slackUploadFile filePath: '/var/jenkins_home/workspace/pipeline-jenkinstest/arachni_report.html.zip', initialComment:  'HEY HEY'
	
           //emailext attachmentsPattern: 'generatedFile.txt',
           //body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
           //recipientProviders: [developers(), requestor()],
           //subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"	
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
    } 
    else {
        color = 'RED'
        colorCode = '#FF0000'
    }

    slackSend(color: colorCode, message: message)
    //slackUploadFile filePath: '/var/jenkins_home/workspace/pipeline-jenkinstest/arachni_report.html.zip', initialComment:  'HEY HEY'
}

// Fetching change set from Git
@NonCPS
def getChangeSet() {
    return currentBuild.changeSets.collect { cs ->
    cs.collect { entry ->
            "* ${entry.author.fullName}: ${entry.msg}"
        }.join("\n")
    }.join("\n")
}
