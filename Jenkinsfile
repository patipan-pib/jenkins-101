pipeline {
  agent any

  environment {
    REPO_URL    = 'git@github.com:github.com/patipan-pib/jenkins-101.git'
    REPO_BRANCH = 'main'
    SSH_VM2_CRED = 'ssh-vm2'
    SSH_VM3_CRED = 'ssh-vm3'
    VM2_USER = 'Test'
    VM2_HOST = '10.162.209.192'
    VM3_USER = 'Pre'
    VM3_HOST = '10.162.209.18'
  }

  stages {
    stage('Unit Test on VM2') {
      steps {
        sshagent(credentials: ["${SSH_VM2_CRED}"]) {
          sh """
            ssh -o StrictHostKeyChecking=no ${VM2_USER}@${VM2_HOST} '
              rm -rf ~/simple-api &&
              git clone -b ${REPO_BRANCH} ${REPO_URL} ~/simple-api &&
              cd ~/simple-api &&
              pip3 install -r requirements.txt &&
              pytest -q
            '
          """
        }
      }
    }

    stage('Robot Test on VM2') {
      steps {
        sshagent(credentials: ["${SSH_VM2_CRED}"]) {
          sh """
            ssh ${VM2_USER}@${VM2_HOST} '
              rm -rf ~/simple-api-robot &&
              git clone https://github.com/YOUR-ORG/simple-api-robot.git ~/simple-api-robot &&
              cd ~/simple-api-robot &&
              robot tests/
            '
          """
        }
      }
    }

    stage('Deploy to VM3') {
      steps {
        sshagent(credentials: ["${SSH_VM3_CRED}"]) {
          sh """
            ssh ${VM3_USER}@${VM3_HOST} '
              rm -rf ~/simple-api &&
              git clone -b ${REPO_BRANCH} ${REPO_URL} ~/simple-api &&
              cd ~/simple-api &&
              pip3 install -r requirements.txt &&
              nohup python3 app.py > app.log 2>&1 &
            '
          """
        }
      }
    }
  }
}
