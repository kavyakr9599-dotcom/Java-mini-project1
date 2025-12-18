pipeline {
    agent none
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    parameters {
      string(name: 'Tomcat_IP', defaultValue: '16.171.197.33', description: 'IP of Sample App')
    }
    stages {
      stage('Checkout Code') {
        agent { label 'slave' }
            steps {
                git branch: 'main',
                    url: 'https://github.com/kavyakr9599-dotcom/Java-mini-project1.git'
            }
        }
  stage('build') {
    agent any
    steps {
      dir('sample-app') {
      sh 'mvn clean package -DskipTests'
      }
    }
  }
    stage('Upload to JFrog') {
      agent any
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds',
                                                 usernameVariable: 'JFROG_USER',
                                                 passwordVariable: 'JFROG_PASS')]) {
                    sh '''
                        echo "Uploading WAR to JFrog..."
                        WAR_FILE=$(ls sample-app/target/*.war)
                        curl -u $JFROG_USER:$JFROG_PASS -T $WAR_FILE \
                        "https://trial0clq38.jfrog.io/artifactory/java-project-generic-local/${JOB_NAME}-${BUILD_NUMBER}-sample.war/${JOB_NAME}-${BUILD_NUMBER}-sample.war"
                    '''
                }
            }
        }
  stage('deploy') {
    agent any
    steps {
      sshagent(credentials: ['tomcat_ssh_key']) {
        sh """
        echo "deploying war to tomcat server"
        WAR_FILE=\$(ls sample-app/target/*.war)
                        FILE_NAME=\$(basename "\$WAR_FILE")

                        # Hardcoded Tomcat Server Details
                        SERVER_IP=${params.Tomcat_IP}
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
    archiveArtifacts artifacts: '**/target/*.war'
  }
  failure {
    echo "job built was a failure"
  }
}

}