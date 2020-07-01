def label = "slave-${UUID.randomUUID().toString()}"

podTemplate(label: label, cloud: 'kubernetes',
        containers: [
        containerTemplate(name: 'jnlp',ttyEnabled: true,image: 'jenkins/jnlp-slave:alpine', imagePullPolicy: 'Always'),
        containerTemplate(name: 'jnlp-mvn',ttyEnabled: true,command: 'cat',image: 'jenkins/jnlp-slave-maven:v1.5', imagePullPolicy: 'Always'),
        containerTemplate(name: 'jnlp-docker',ttyEnabled: true,command: 'cat',image: 'jenkins/jnlp-slave-docker:v1.6', imagePullPolicy: 'Always')
    ],
    serviceAccount:'gitops-jenkins',
    volumes: [
	hostPathVolume(mountPath: '/root/.m2', hostPath: '/var/run/m2'),
	hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
    ]) {
   node(label) {
      stage('checkout'){
        container('jnlp') {
          echo "1.Clone Repo Stage"
          git credentialsId: 'GitHubAccess', url: 'https://github.com/successfuljian/app-repo'
          script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            repo_name = '232660966648.dkr.ecr.ap-northeast-2.amazonaws.com'
            app_name = 'gitops-app-demo'
          }
        }
    }
     stage('Build Image'){
        container('jnlp-docker'){   
        echo "2.Build Docker Image Stage test"
        sh "docker build --network host -t ${repo_name}/${app_name}:latest ."
        sh "docker tag ${repo_name}/${app_name}:latest ${repo_name}/${app_name}:${build_tag}"
        }  
     }
     stage('Push Image') {
       container('jnlp-docker'){
       echo "3.Push Docker Image Stage"
       withDockerRegistry(credentialsId: 'ecr:ap-northeast-2:AWS-AKSK', url: 'https://232660966648.dkr.ecr.ap-northeast-2.amazonaws.com/gitops-app-demo') {
        sh "docker push ${repo_name}/${app_name}:latest"
        sh "docker push ${repo_name}/${app_name}:${build_tag}"
       }
      }
     }
   }
}     	        

