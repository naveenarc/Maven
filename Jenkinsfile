pipeline{
    agent any
    tools{
        jdk 'myjava'
        maven 'mymaven'
    }
    environment{
        APP_NAME='java-mvn-app'
        ANSIBLE_SERVER="ec2-user@172.31.42.166"
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
    }
    parameters{
        choice(name:'VERSION',choices:['1.1.0','1.2.0','1.3.0'],description:'version of the code')
        booleanParam(name: 'executeTests',defaultValue: true,description:'tc validity')
    }
    stages{
        stage("COMPILE"){          
            steps{
                script{
                    echo "Compiling the code"
                    sh 'mvn compile'
                }
            }
        }
        stage("UNITTEST"){           
            when{
                expression{
                    params.executeTests == true
                }
            }
            steps{
                script{
                    echo "Testing the code"
                    sh 'mvn test'
                }
            }
            post{
                always{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
         stage("PACKAGE"){
                     steps{
                script{
                    echo "Packaging the code"
                    sh 'mvn package'
                }
            }
        }       
         stage("copy ansible files from jenkins server to ACM"){
             steps{
                 script{
                     echo "Copying ansible files from Jenkins server to ACM"

                     sshagent(['deploy-server-key']) {
                sh "scp -o StrictHostKeyChecking=no ansible/* ${ANSIBLE_SERVER}:/home/ec2-user"
    withCredentials([sshUserPrivateKey(credentialsId: 'ansible-target-key',keyFileVariable: 'keyfile',usernameVariable: 'user')]){
                       sh 'scp $keyfile $ANSIBLE_SERVER:/home/ec2-user/.ssh/id_rsa'
                       }
}
                 }
             }
         }
         stage("execute ansible playboooks"){
             steps{
                 script{
                     echo "running ansible playbook"
                sshagent(['deploy-server-key']) {
                sh "scp -o StrictHostKeyChecking=no ./prepare-ACM.sh ${ANSIBLE_SERVER}:/home/ec2-user"
                sh "ssh -o StrictHostKeyChecking=no ${ANSIBLE_SERVER} bash /home/ec2-user/prepare-ACM.sh"
                 }
             }
         }
    }
 }
}