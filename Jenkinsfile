ECR_URI = "191845259489.dkr.ecr.ap-northeast-2.amazonaws.com/your-repo-name"
APP_NAME = "flask-dev2"

node {
     try{
          stage('Clone repository') {
               checkout scm
          }
          /*
          stage('Initialize Docker'){
               def dockerHome = tool 'myDocker'
               env.PATH = "${dockerHome}/bin:${env.PATH}"
          }
          stage('Maven Build') {
               sh('ls')
               sh('chmod +x ./mvnw')
               sh('./mvnw package -Dmaven.repo.local=/var/jenkins_home/.m2/repository')               
          }
          */
          
         docker.image('maven:3.8.1-adoptopenjdk-11').inside('-v $HOME/.m2:/root/.m2') {
             stage('Build') {
                 sh 'mvn -B'
             }
         }
          stage('Build image') {
               //sh('docker build -t sample-java --network=host .')
               sh('docker images')
          }
          stage('Push image') {
               //ecr_push()
          }
       /*
          stage('Clone and Push manifest') {
               //git Clone
               def gitValues = git branch: 'main', credentialsId: 'e347081a-3fef-4da4-ad62-bd7eca486575', url:'git@github.com:nauco/ops.git'
               modify_manifest('deployment.yaml')
               modify_manifest('./prod/deployment.yaml')
               sh('git config --global user.email "hodong42@gmail.com"') 
               sh('git config --global user.name "hodong"')
               sh('git add .')
               sh('git commit -m "update image"')
               sshagent(['e347081a-3fef-4da4-ad62-bd7eca486575']) {
                    sh "git push origin main"
               }
          }
          stage('Trigger Argocd and Deploy') {
               deploy_app()
          }
          slackSend (channel: '#project', color: 'good', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
          */
     }
     catch (e) {
          //slackSend (channel: '#project', color: 'danger', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")    
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

def modify_manifest(path) {
     def filename = path
     def data = readYaml file: filename
     data.spec.template.spec.containers[0].image = "${ECR_URI}:${env.BUILD_NUMBER}"
     sh "rm $filename"
     writeYaml file: filename, data: data
}

def deploy_app() {
     sh('curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.0.1/argocd-linux-amd64')
     sh('chmod +x /usr/local/bin/argocd')
     withCredentials([string(credentialsId: "argo-pw", variable: 'ARGOCD_PASSWORD')]) {
          sh('argocd login 10.100.216.26:80 --username admin --password=$ARGOCD_PASSWORD --grpc-web --insecure')
          sh """
          if argocd app list -o name | grep -x '${APP_NAME}' ; then
              echo 'already exist'
          else
              argocd app create ${APP_NAME} --repo https://github.com/nauco/ops.git --path ./ --dest-namespace default --dest-server https://kubernetes.default.svc --sync-policy auto
              argocd app patch ${APP_NAME} --patch '{"metadata": {"annotations": {"notifications.argoproj.io/subscribe.on-sync-succeeded.slack":"project"}}}' --type merge
          fi
          """
     }
}
