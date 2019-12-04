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
		    // Get the Maven tool.
                    //  NOTE: This 'M3' Maven tool must be configured
                    //        in the global configuration.
                    echo 'Pulling.......' + env.BRANCH_NAME
                    //def mvnHome = tool 'maven-3.3.9'
                    //if (isUnix()) {
                       # def targetVersion = getDevVersion()
                        #print 'target build version...'
                        #print targetVersion
			
			     //sh 'mvn clean compile'
			    sh "mvn -Dintegration-tests.skip=true -Dbuild.number=${targetVersion} clean package"
			    // get the current development version
                        #developmentArtifactVersion = "${pom.version}-${targetVersion}"
                       # print pom.version
		    }
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

def getDevVersion() {
    def gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def versionNumber;
    if (gitCommit == null) {
        versionNumber = env.BUILD_NUMBER;
    } else {
        versionNumber = gitCommit.take(8);
    }
    print 'build  versions...'
    print versionNumber
    return versionNumber
}
