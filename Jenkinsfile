def label = "slave-${UUID.randomUUID().toString()}"

podTemplate(label: label, cloud: 'kubernetes',
        containers: [
        containerTemplate(name: 'jenkins-slave',image: '232660966648.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins-slave:latest',ttyEnabled: true,command: 'cat')
    ],
    serviceAccount:'gitops-jenkins',
    volumes: [
	hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
    ]) {
   node(label) {
     container('jenkins-slave'){
      stage('checkout'){
          echo "1.Clone Repo Stage"
          git credentialsId: 'GitHubAccess', url: 'https://github.com/successfuljian/app-repo'
          script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            repo_name = '232660966648.dkr.ecr.ap-northeast-2.amazonaws.com'
            app_name = 'gitops-app-demo'
          }
    }
     stage('Build Image'){
        echo "2.Build Docker Image Stage test"
        sh "docker build --network host -t ${repo_name}/${app_name}:latest ."
        sh "docker tag ${repo_name}/${app_name}:latest ${repo_name}/${app_name}:${build_tag}"
     }
     stage('Push Image') {
       echo "3.Push Docker Image Stage"
       withDockerRegistry(credentialsId: 'ecr:ap-northeast-2:AWS-AKSK', url: 'https://232660966648.dkr.ecr.ap-northeast-2.amazonaws.com/gitops-app-demo') {
        sh "docker push ${repo_name}/${app_name}:latest"
        sh "docker push ${repo_name}/${app_name}:${build_tag}"
       }
     }
   }
 }     	        
}
