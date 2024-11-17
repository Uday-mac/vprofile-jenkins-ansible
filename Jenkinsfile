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
      NEXUSIP = '44.220.143.177'
      NEXUSPORT ='8081'
      sonar_scanner = 'sonar4.7'
      sonar_server = 'sonar'
      NEXUS_LOGIN = 'nexus_login'
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
              sh ''' ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
              -Dsonar.projectName=vprofile-repo \
              -Dsonar.projectVersion=1.0 \
              -Dsonar.source=src \
              -Dsonar.java.binaries=target/classes/com/visualpathit/account/controller \
              -Dsonar.junit.reportsPath=target/surefire-reports \
              -Dsonar.jacoco.reportPaths=target/jacoco.exec \
              -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
              '''
            }
          }
        }
        stage('build artifact') {
          steps {
            sh 'mvn -s settings.xml -DskipTests install'
          }
        }

        stage("upload to nexus") {
          steps {
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                groupId: 'QA',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: "${RELEASE_REPO}" , 
                credentialsId: "${NEXUS_LOGIN}",
                artifacts: [
                    [artifactId: 'vproapp',
                    classifier: '',
                    file: 'target/vprofile-v2.war',
                    type: 'war']
                ]
                )
          }
        }

        stage("deploy using ansible") {
          steps {
            ansiblePlaybook([
              playbook: 'ansible/sites.yml',
              inventory: 'ansible/satge-inventory',
              credentialsId: 'app_login',
              colorized: true,
              installation: 'ansible',
              disableHostKeyChecking: true,
              extraVars: [
                NEXUS_USER = 'admin',
                NEXUX_PASS = 'admin',
                NEXUX_IP = '44.220.143.177',
                NEXUS_PORT = '8081',
                RELEASE_REPO = 'vprofile-release',
                groupId = 'QA',
                artifactId = 'vproapp',
                version = "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}"
              ]
            ])
          }
        }
    }
    post {
      always {
        slackSend channel: '#jenkins-ansible',
        color: color_map[currentBuild.currentResult],
        message: "Find Status of Pipeline:- ${currentBuild.currentResult}: ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n more info at ${BUILD_URL}"
      }
    }
}