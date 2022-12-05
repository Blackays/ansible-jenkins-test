def gv

pipeline {
    agent any
    environment {
        DROPLET_PUBLIC_IP = "167.99.136.157"
    }
    stages {
        stage("Copy files to ansible server") {
            steps {
                script {
                    echo "copying all neccessary files to ansible control node"
                    def droplet = "root@${DROPLET_PUBLIC_IP}"
                    sshagent(['droplet-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no ansible/* ${droplet}:/root"                    
                        withCredentials([sshUserPrivateKey(credentialsId: 'droplet', keyFileVariable: 'keyfile',usernameVariable: 'user')]) {
                            sh 'scp $keyfile ${droplet}:/root/ssh-key.pem'  //single quotes and no brackats {} for variables
                        }
                    }                    
                }
            }
        }
        stage("Execute ansible playbook") {
            steps {
                script {
                    echo "Calling ansible playbook to configure instances"
                    def remote = [:]
                    remote.name = "ansible-server"
                    remote.host = DROPLET_PUBLIC_IP
                    remote.allowAnyHosts = true

                    withCredentials([sshUserPrivateKey(credentialsId: 'droplet', keyFileVariable: 'keyfile',usernameVariable: 'user')]) {
                        remote.user = user
                        remote.identityFile = keyfile
                        sshCommand remote: remote, command: "ansible-playbook playbook.yaml"
                    }
                }
            }
        }
    }   
}