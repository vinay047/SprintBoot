pipeline {
   agent any

      parameters{
          string(name: 'ECS_CLUSTER_NAME',
          defaultValue: 'terraform-eks-demo',
          description: 'ECS_CLUSTER_NAME')

          string(name: 'JUMP_SERVER_IP',
          defaultValue: '54.204.69.201',
          description: 'JUMP_SERVER_IP')

          string(name: 'SONAR_IP',
          defaultValue: '54.210.132.148',
          description: 'JUMP_SERVER_IP')

          string(name: 'JENKINS_ACCESS_KEY_ID',
          defaultValue: 'JENKINS_ACCESS_KEY_ID',
          description: 'JENKINS_ACCESS_KEY_ID')

          password(name: 'JENKINS_SECRET_ACCESS_KEY',
          defaultValue: 'JENKINS_SECRET_ACCESS_KEY',
          description: 'JENKINS_SECRET_ACCESS_KEY')

          string(name: 'DEFAULT_REGION',
          defaultValue: 'us-east-1',
          description: 'DEFAULT_REGION')
       }




   stages {

    stage('Checkout') {
         steps {
        git 'https://github.com/vinay047/SprintBoot'

        }
       }
      stage('Build') {


         steps {

            withMaven(maven: 'mvn3_6'){
                sh 'mvn install -Dsonar.skip=true '
            }

         }

      }

      stage('Unit Test') {


               steps {

                  withMaven(maven: 'mvn3_6'){
                      sh 'mvn test'
                  }

               }
      }

      stage('White Source') {


                     steps {

                        withMaven(maven: 'mvn3_6'){
                            sh 'mvn whitesource:update'
                        }

                     }
            }



      stage('Build image & upload') {
                 steps {
                    script {
                     def customImage = docker.build("sridharkidambi/skimages")
                     docker.withRegistry('https://registry.hub.docker.com/','dockerhub'){
                        customImage.push()
                      }
                    }

                 }
      }



      stage('Deploy to K8s') {

                       steps {

                          sshagent(['jenkins']){
                            sh "scp -o StrictHostKeyChecking=no pods.yml services.yml  ec2-user@${JUMP_SERVER_IP}:/home/ec2-user/"
                            script {
                                try{
                                sh 'rm -rf  ~/.aws/credentials'
                                sh 'rm -rf  ~/.aws/config'
                                sh 'aws configure set aws_access_key_id ${JENKINS_ACCESS_KEY_ID}'
                                sh 'aws configure set aws_secret_access_key ${JENKINS_SECRET_ACCESS_KEY}'
                                sh 'aws configure set default.region ${DEFAULT_REGION}'
                                sh 'ssh ec2-user@${JUMP_SERVER_IP} aws eks --region us-east-1 update-kubeconfig --name ${ECS_CLUSTER_NAME}'
                                sh 'ssh ec2-user@${JUMP_SERVER_IP} ls -latr'
                                sh 'ssh ec2-user@${JUMP_SERVER_IP} kubectl apply -f pods.yml'
                                 sh 'ssh ec2-user@${JUMP_SERVER_IP} kubectl apply -f services.yml' 
                                }catch(error){
                                    sh 'ssh ec2-user@${JUMP_SERVER_IP} kubectl create -f .'
                                }
                            }
                          }

                       }
            }
  }

}
