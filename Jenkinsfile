def color_map = [
  'SUCCESS': 'good',
  'FAILURE': 'danger',
]

pipeline {
    agent any 
    tools {
      jdk 'OracleJDK8'
      maven 'MAVEN3'
    }
    environment {
      NEXUS_USER = 'admin'
      NEXUS_PASS = 'admin'
      SNAP_REPO = 'vprofile-snap'
      RELEASE_REPO = 'vprofile-release'
      CENTRAL_REPO = 'vprofile-mvn-central'
      NEXUS_GRP_REPO = 'vprofile-group'
      NEXUSIP = '52.73.146.126'
      NEXUSPORT ='8081'
      sonar_scanner = 'sonar4.7'
      sonar_server = 'sonar'
    }
    stages {
        stage("Test") {
            steps {
              sh 'mvn -s settings.xml test'
            }
        }

        stage("Checkstyle Analysis") {
            steps {
              sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage("sonar analysis") {
          environment {
            scannerHome = tool "${sonar_scanner}"
          }
          steps {
            withSonarQubeEnv("${sonar_server}") {
            sh ''' ${scannerHome}/bin/sonar-scanner" -Dsonar.projectKey=vprofile \
            -Dsonar.projectName=vprofile \
            -Dsonar.projectVersion=1.0 \
            -Dsonar.source=src \
            -Dsonar.java.binaries=target/classes/com/visualpathit/account/controller \
            -Dsonar.junit.reportsPath=target/surefire-reports \
            -Dsonar.jacoco.reportPaths=target/jacoco.exec \
            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml '''
          }
        }

        stage('build artifact') {
          steps {
            sh 'mvn -s settings.xml -DskipTests install'
          }
        }
    }
    post {
      always {
        slackSend channel: '#jenkins-ansible',
        color: color_map[currentBuild.currentResult]
        message: "Find Status of Pipeline:- ${currentBuild.currentResult}: ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n more info at ${BUILD_URL}"
      }
    }
}