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
                echo 'Clean Build'
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing'
                sh 'mvn test'
            }
        }
        stage('sonar'){
            echo 'Sonar Scanner'
		    sh "mvn sonar:sonar -Dsonar.host.url=http://198.198.10.46:9000"
   }
}
}
