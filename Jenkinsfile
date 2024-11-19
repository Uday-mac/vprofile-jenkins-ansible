def color_map = [
  'SUCCESS': 'good',
  'FAILURE': 'danger',
]

pipeline {
    agent any 
    environment {
     NEXUS_PASS = credentials('nexus_pass')
    }
    stages {
        stage("providing parameters") {
          steps {
            script {
              properties {
                parameters ([
                  string(
                    defaultValue:'',
                    name: 'BUILD', 
                  ),
                  string(
                    defaultValue: '',
                    name: 'TIME',
                  )
                ])
              }
            }
          }
        }
        stage("deploy using ansible") {
          steps {
            ansiblePlaybook(
              playbook: 'ansible/sites.yml',
              inventory: 'ansible/prod-inventory.ini',
              credentialsId: 'app_login',
              colorized: true,
              installation: 'ansible',
              disableHostKeyChecking: true,
              extraVars: [
                USER: 'admin',
                PASS: 'admin',
                IP: '98.80.123.89',
                PORT: '8081',
                REPO: 'vprofile-release',
                groupId: 'QA',
                artifactId: 'vproapp',
                version: "${env.BUILD}-${env.TIME}"
              ]

            )
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