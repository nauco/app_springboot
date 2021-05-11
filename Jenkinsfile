ECR_URI = "your-ecr-repo-name"
APP_NAME = "flask-dev2"

node {
     stage('Approval') {
          try {
              echo "========== create slack message ========="
              def cmd = """
                  curl -d '{"buildNumber": "${BUILD_NUMBER}", "jobName": ${JOB_NAME}, "channel": "project"}' \
                  -H "Content-Type: application/json" \
                  -X POST https://01zsmguyhh.execute-api.ap-northeast-2.amazonaws.com/agw/create-slack-message
              """
              def result = getShellCommandResult(cmd)
              def resultJson = readJSON text: "${result}"
              if (resultJson['status'] == 'Success') {
                  print('Created Slack Message!')
              } else if (resultJson['status'] == 'Fail') {
                  throw new Exception('A fail was returned from the api call...')
              } else {
                  throw new Exception('An incorrect status value was returned...')
              }
               
               
              def approval = input(
                  id: 'wait-approval',
                  message: 'Approve?',
                  submitterParameter: 'approver',
                  parameters: [choice(choices: ['Cancel', 'Deploy'],description: 'Are you sure?',name: 'choice')]
                  )
               if (approval['choice'] == 'Deploy') {
                 print('choice deploy')
                 currentBuild.result = 'Success'
               } else {
                 throw new Exception('Choose cancel')
                }
          } catch(Exception e) {
              error e
              currentBuild.result = 'Fail'
          }   
     }
     stage('Clone repository') {
          checkout scm
     }
     stage('Initialize Docker'){
          def dockerHome = tool 'myDocker'
          env.PATH = "${dockerHome}/bin:${env.PATH}"
     }
     stage('Maven Build') {
          sh('chmod +x ./mvnw')
          sh("./mvnw package -Dmaven.repo.local=/root/.m2/repository")               
     }
     stage('Build image') {
          sh('docker build -t sample-java --network=host .')
     }
     stage('Push image') {
          ecr_push()
     }
}

def getShellCommandResult(cmd) {
    return sh(script: cmd, returnStdout: true).trim()
}

def ecr_push() {
     withAWS(credentials: 'ecr-credit', region: 'ap-northeast-2') {
          def login = ecrLogin()
          sh "${login}"
          sh("docker tag sample-java:latest ${ECR_URI}:${env.BUILD_NUMBER}")
          sh("docker push ${ECR_URI}:${env.BUILD_NUMBER}")
     }
}





    
