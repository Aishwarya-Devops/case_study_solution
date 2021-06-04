 pipeline {
     
  agent any
  tools
  {
      'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker'
  }

  environment {
        def mavenHome = tool name:'maven 3',type: 'maven'
        def mavenCMD ="${mavenHome}/bin/mvn"
        def docker = tool name:'docker',type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        def dockerCMD=" $docker/bin/docker"
        registry = "aishwaryadevops/app-image"
        registryCredential = 'dockerhub'
        def dockerImage = ''
  }
  
 stages{
  
    stage('code checkout'){
        steps{
        git 'https://github.com/Aishwarya-Devops/batch10.git'
        }
    }
    stage('build_test_package'){
        steps{
    sh "${mavenCMD} clean package"
    }
    post {
        success {
          // publish html
          publishHTML target: [
              allowMissing: false,
              alwaysLinkToLastBuild: false,
              keepAll: true,
              reportDir: 'target',
              reportFiles: 'index.html',
              reportName: 'RCov Report'
            ]
        }
      }
    }
      stage('SonarQube analysis') {
            steps {
               
 script {
     try{
       def scannerHome = tool 'sonarqube';
           withSonarQubeEnv(installationName: 'SonarQube') {
           sh "${tool("sonarqube")}/bin/sonar-scanner \
           -Dsonar.projectKey=test-node-js \
           -Dsonar.sources=. \
           -Dsonar.css.node=. \
           -Dsonar.host.url=http://35.235.81.148:9000 \
           -Dsonar.login=admin \
           -Dsonar.password=sonar\
           -Dsonar.java.binaries=./target/classes"

               }

           }
            
       catch(Exception e) {
   
        error "sonar analysis failed, please read logs..."
        }
     }

    
       } 

      
          
      }
    stage('Build Docker image'){
        steps{
          echo "Building the docker image for application.."
          
        //   sh "docker run -p 8080:8080 --user root \
        //        -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean"
         script
          {
              try{
            dockerImage = docker.build registry + ":$BUILD_NUMBER"
              }
               catch(Exception e) {
   
                error "docker build failed, please read logs..."
              }
          }
        
          
         // sh "sudo ${dockerCMD} build -t aishwaryadevops/app-image:$BUILD_NUMBER ."
        }
    }
        stage('Push Docker image'){
        steps{
          echo "Pushing the docker image for application.."
          
        //   sh "docker run -p 8080:8080 --user root \
        //        -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean"
         script
          {
              try{
               docker.withRegistry( '', registryCredential ) { 
                dockerImage.push() 
               }
              }
               catch(Exception e) {
   
               error "docker push failed, please read logs..."
                }
          }
        
          
         // sh "sudo ${dockerCMD} build -t aishwaryadevops/app-image:$BUILD_NUMBER ."
        }
        
    }

    stage ("configure and deploy to another instance")
    {
      
        steps{
            echo "configure and deploy the image to another instance.."
            ansiblePlaybook credentialsId: 'ssh-key', disableHostKeyChecking: true , extraVars: [tagname : "${BUILD_NUMBER}"], installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-playbook.yml'
        }
    }
     stage('Remove Unused docker image') {
      steps{
      sh "docker rmi $registry:$BUILD_NUMBER"
  }
}
 
    
 }
 
 post{
  failure {  
             emailext attachLog: true, body: 'Please check build failure', subject: 'Bad Build', to: 'aiwspace@gmail.com'  
          }  
  success{
           emailext attachLog: true, body: 'Build succeeded', subject: 'Success Build', to: 'aiwspace@gmail.com'  
       
          }
 }
 

  
 }
 
