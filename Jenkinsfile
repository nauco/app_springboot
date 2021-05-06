ECR_URI = "191845259489.dkr.ecr.ap-northeast-2.amazonaws.com/your-repo-name"
APP_NAME = "flask-dev2"

node {
     stage('Approval') {
               try {
                   def approval = input(
                       id: 'wait-approval',
                       message: 'Approve?',
                       submitterParameter: 'approver',
                       parameters: [choice(choices: ['Cancel', 'Deploy'],description: 'Are you sure?',name: 'choice')]
          )
             if (approval['approver'] != "${administrator}") {
                 throw new Exception('You do not have permission.')
             }

                   if (approval['choice'] == 'Deploy') {
                 print('choice deploy')
                 currentBuild.result = 'Success'
             } else {
                 throw new Exception('Choosed cancel')
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

def ecr_push() {
     withAWS(credentials: 'ecr-credit', region: 'ap-northeast-2') {
          def login = ecrLogin()
          sh "${login}"
          sh("docker tag sample-java:latest ${ECR_URI}:${env.BUILD_NUMBER}")
          sh("docker push ${ECR_URI}:${env.BUILD_NUMBER}")
     }
}





    
