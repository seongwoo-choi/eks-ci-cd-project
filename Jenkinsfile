pipeline {
  agent any
  tools {
    maven 'Maven3' 
  }
  
  environment {
    dockerHubRegistry = 'how0326/tomcat'
    dockerHubRegistryCredential = 'docker'
    githubCredential = 'git_hub'
    gitEmail = 'how0326@gmail.com'
    gitName = 'seongwoo-choi'
  }
  
  stages {
   
    stage('Checkout Application Git Branch') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: githubCredential, url: 'https://github.com/mini-cicd-project/mini-cicd-project.git']]])
      }
      post {
        failure {
          echo 'Repository clone failure' 
        }
        success {
          echo 'Repository clone success' 
        }
      }
    }
    
    stage('Maven Jar Build') {
      steps {
        sh 'mvn clean install'  
      }
      post {
        failure {
          echo 'Maven war build failure' 
        }
        success {
          echo 'Maven war build success'
        }
      }
    }
    
    
    stage('Docker Image Build') {
      steps {
        sh "docker build . -t ${dockerHubRegistry}:${currentBuild.number}"
        sh "docker build . -t ${dockerHubRegistry}:latest"
      }
      post {
        failure {
          echo 'Docker image build failure'
          slackSend (color: '#FF0000', message: "FAILED: Docker Image Build '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        success {
          echo 'Docker image build success'
          slackSend (color: '#0AC9FF', message: "SUCCESS: Docker Image Build '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
      }
    }  
    
    stage('Docker Image Push') {
      steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker push ${dockerHubRegistry}:latest"
          
          sleep 10
        } 
      }
      post {
        failure {
          echo 'Docker Image Push failure'
          sh "docker rmi ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker rmi ${dockerHubRegistry}:latest"
          slackSend (color: '#FF0000', message: "FAILED: Docker Image Push '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        success {
          echo 'Docker Image Push success'
          sh "docker rmi ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker rmi ${dockerHubRegistry}:latest"
          slackSend (color: '#0AC9FF', message: "SUCCESS: Docker Image Push '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
      }
    }
    
    stage('K8S Manifest Update') {
      steps {
        git credentialsId: githubCredential,
            url: 'https://github.com/mini-cicd-project/mini-cicd-project.git',
            branch: 'main'  
       
        sh "git config --global user.email ${gitEmail}"
        sh "git config --global user.name ${gitName}"
        sh "sed -i 's/tomcat:.*/tomcat:${currentBuild.number}/g' deploy/production.yaml"
        sh "git add ."
        sh "git commit -m 'fix:${dockerHubRegistry} ${currentBuild.number} image versioning'"
        sh "git branch -M main"
        sh "git remote remove origin"
        sh "git remote add origin git@github.com:mini-cicd-project/mini-cicd-project.git"
        sh "git push -u origin main"
      }
      post {
        failure {
          echo 'K8S Manifest Update failure'
          slackSend (color: '#FF0000', message: "FAILED: K8S Manifest Update '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        success {
          echo 'K8s Manifest Update success'
          slackSend (color: '#0AC9FF', message: "SUCCESS: K8S Manifest Update '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
      }
    }
    
  }
}
