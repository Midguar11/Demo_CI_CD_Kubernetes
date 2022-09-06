node {
    
    stage("Git Clone"){

        git credentialsId: 'e55b3f41-c210-44c2-81d9-234977265fd1', url: 'https://github.com/Midguar11/Demo_CI_CD_Kubernetes.git'
        sh 'sudo chmod 700 gradlew' 
    }
    
    stage('Sonarqube Scan') {
        withSonarQubeEnv('sonarqube') {
        sh "./gradlew sonarqube -D sonar.projectKey=Demo_CI_CD_Kubernetes"
        }
    }
 
    stage("Quality Gate"){
        timeout(time: 5, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true    
        }
    }
    
    
    stage('Gradle Build') {                   
       sh './gradlew build'
    }
    
    stage("Docker build"){
        sh 'docker version'
        sh 'docker build -t jhooq-docker-demo .'
        sh 'docker tag jhooq-docker-demo midguar/jhooq-docker-demo:jhooq-docker-demo'
    }
    
    stage("Docker Hub Login"){
        withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'PASSWORD')]) {
            sh 'docker login -u midguar -p $PASSWORD'
        }
    } 
    
    stage("Push Image to Docker Hub"){
        sh 'docker push midguar/jhooq-docker-demo:jhooq-docker-demo'
    }
    
    stage("SSH Into Kubernetes Server with secret key") {
        def remote = [:]
        remote.name = 'KubeMaster'
        remote.host = '192.168.0.124'
        remote.allowAnyHosts = true
        
        node {
            withCredentials([sshUserPrivateKey(credentialsId: 'KubeMasterSSHKeyPem', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                remote.user = userName
                remote.identityFile = identity
                stage('Put deploymentservice.yaml into KubeMaster') {
                    sshPut remote: remote, from: '/var/lib/jenkins/workspace/Demo_App/Demo_Kubernetes/deploymentservice.yaml', into: '.'
                }
                
                stage('Deploy spring boot') {
                  sshCommand remote: remote, command: 'kubectl apply -f deploymentservice.yaml'
                }
                
                
            }
        }
    }
    
    stage("Finished Build Send email to Admin"){
        emailext attachLog: true, body: 'DemoApp2 Kubernetes Build Completed', subject: 'DemoApp2_Kubernetes_pipeline', to: 'szendiattila11@gmail.com'
    }

   stage('The countdown has started. The Demo Webshop will be deleted after 15 minutes'){
    echo 'http://demoapp2.devopsempire.com/'
  }

}
