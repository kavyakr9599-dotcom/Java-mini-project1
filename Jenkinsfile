pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    stages {
      stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kavyakr9599-dotcom/Java-mini-project1.git'
            }
        }
  stage('build') {
    steps {
      dir('sample-app') {
      sh 'mvn clean package -DskipTests'
      }
    }
  }
  stage('deploy') {
    steps {
      sshagent(credentials: ['tomcat_ssh_key']) {
        sh """
        echo "deploying war to tomcat server"
        WAR_FILE=\$(ls sample-app/target/*.war)
                        FILE_NAME=\$(basename "\$WAR_FILE")

                        # Hardcoded Tomcat Server Details
                        SERVER_IP=13.60.210.40
                        SERVER_USER=ubuntu
                        TOMCAT_DIR=/opt/tomcat/webapps

                        echo "Using server IP: \$SERVER_IP"

                        # Copy WAR file to remote /tmp
                        scp -o StrictHostKeyChecking=no "\$WAR_FILE" \$SERVER_USER@\$SERVER_IP:/tmp/

                        # Move WAR file to Tomcat's webapps directory
                        ssh -o StrictHostKeyChecking=no \$SERVER_USER@\$SERVER_IP "sudo mv /tmp/\$FILE_NAME \$TOMCAT_DIR/"

                        # Restart Tomcat
                        ssh -o StrictHostKeyChecking=no \$SERVER_USER@\$SERVER_IP "sudo systemctl restart tomcat"

                        echo "Deployment completed successfully!"
                    """
      }
    }
  }

}
post {
  success {
    echo "job built successfully"
    archiveArtifacts artifacts: '*/target/.war'
  }
  failure {
    echo "job built was a failure"
  }
}

}