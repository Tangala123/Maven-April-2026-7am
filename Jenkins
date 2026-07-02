pipeline {
    agent any

    stages {
        stage('checkout') {
            steps {
               git branch: 'main', credentialsId: 'GITHUB', url: 'https://github.com/Tangala123/Maven-April-2026-7am.git'
            }
        }
        stage('build'){
            steps{
                sh "mvn clean package"
            }
        }
        stage('sonar-qube') {
            steps{
                 withSonarQubeEnv('sonar-9') { 
                    sh "mvn sonar:sonar"
                }
            }
        }
        // stage("sonarquality"){
        //     steps{
        //         script{ 
        //             def qg = waitForQualityGate()
        //             if (qg.status != 'ok'){
        //                 error "status failed"
        //             }
        //         } 
        //     }
        // }
        stage("Nexus-upload"){
            steps{
                // nexusArtifactUploader artifacts: [[artifactId: 'DevOps', classifier: '', file: 'target/DevOps.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.google', nexusUrl: '172.31.10.166:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'release-version', version: '1.1'
                script{
                  def pom = readMavenPom file: 'pom.xml'
                  def version = pom.version
                  def repoName = version.endsWith("SNAPSHOT") ? "snapshot-version" : "release-version"
                  nexusArtifactUploader artifacts: [[artifactId: 'DevOps', classifier: '', file: 'target/DevOps.war', type: 'war']], 
                  credentialsId: 'nexus',
                  groupId: 'com.google', 
                  nexusUrl: '172.31.10.166:8081',
                  nexusVersion: 'nexus3',
                  protocol: 'http', 
                  repository: "${repoName}", 
                  version: "${version}"
              }
            }
        }
        stage("docker build"){
            steps{
                sh "docker build -t tangalalakshmi/myjava-app:v1 ."
            }
        }
        stage ("docker login"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'pwd', usernameVariable: 'usr')]) {
                sh "docker login -u ${usr} -p ${pwd}"
                }
                
            }
        }
        stage("docker push") {
            steps{
                sh "docker push tangalalakshmi/myjava-app:v1"
            }
        }
        stage("Deployment"){
            steps{
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                sh """
                        aws eks --region us-east-1 update-kubeconfig --name EKS 
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                        kubectl get deploy
                        kubectl get svc
                    """
                }
            }
        }
    }
}
