pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps {
               git 'https://github.com/rishipollai/dockerJenkins.git'
            }
        }
   
        stage('Sending Dockerfile to Ansible server over ssh') {
            steps {
              sshagent(['ansible_demo']) {
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115'
                 sh 'scp /var/lib/jenkins/workspace/pipeline_demo/* ubuntu@3.109.56.115:/home/ubuntu/'
               }
            }
         }
         
        stage('Dockerfile build image') {
            steps {
              sshagent(['ansible_demo']) {
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 cd /home/ubuntu/'
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 docker image build -t $JOB_NAME:V1.$BUILD_ID .'
               }
            }
         }
         
        stage('Docker tag images') {
            steps {
              sshagent(['ansible_demo']) {
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 cd /home/ubuntu/'
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 docker image tag $JOB_NAME:V1.$BUILD_ID rishipollai/$JOB_NAME:V1.$BUILD_ID'
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 docker image tag $JOB_NAME:V1.$BUILD_ID rishipollai/$JOB_NAME:latest'
               }
            }
         }
         
        stage('Push Docker images to dockerhub') {
            steps {
              sshagent(['ansible_demo']) {
                  withCredentials([string(credentialsId: 'dockerhub_password', variable: 'dockerhub_password')]) {
                     sh "ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 docker login -u rishipollai -p ${dockerhub_password}"
                     sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 docker image push rishipollai/$JOB_NAME:V1.$BUILD_ID'
                     sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 docker image push rishipollai/$JOB_NAME:latest'
                     
                     sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 docker image rm rishipollai/$JOB_NAME:V1.$BUILD_ID rishipollai/$JOB_NAME:latest $JOB_NAME:V1.$BUILD_ID'
                 }
                
               }
            }
         }
         
         stage('Copy file from ansible to kubernetes server') {
            steps {
              sshagent(['ansible_demo']) {
                  
                     sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.111.38.241'
                     sh 'scp /var/lib/jenkins/workspace/pipeline_demo/* ubuntu@3.111.38.241:/home/ubuntu/'
                
            }
         }
       }
       
         stage('Kubernetes deployement using ansible') {
            steps {
              sshagent(['ansible_demo']) {
                  
                     sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 cd /home/ubuntu/'
                     sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.56.115 sudo ansible-playbook ansible.yml'
                
            }
         }
       }
    }

}
