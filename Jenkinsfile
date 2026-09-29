```groovy
pipeline {
    agent any

    parameters {
        string(
            name: 'TOMCAT_SERVER_IP',
            defaultValue: '',
            description: 'Enter the Tomcat server IP address'
        )
    }

    tools {
        jdk 'Java-21'
        maven 'Maven-3.9.9'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Kushbalaji/suprith_Jenkins_Assignment.git'
            }
        }

        stage('Build') {
            steps {
                dir('sample-app') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Upload to JFrog') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'jfrog-creds',
                        usernameVariable: 'JFROG_USER',
                        passwordVariable: 'JFROG_PASS'
                    )
                ]) {
                    sh '''
                        echo "Uploading WAR to JFrog..."

                        WAR_FILE=$(ls sample-app/target/*.war)

                        curl -u "$JFROG_USER:$JFROG_PASS" \
                        -T "$WAR_FILE" \
                        "https://triald13vww.jfrog.io/artifactory/javarepo/${JOB_NAME}-${BUILD_NUMBER}-sample.war"
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(credentials: ['tomcat-ssh-key']) {
                    sh '''
                        echo "Deploying WAR to Tomcat server..."

                        echo "Tomcat Server IP: ${TOMCAT_SERVER_IP}"

                        WAR_FILE=$(ls sample-app/target/*.war)

                        SERVER_IP="${TOMCAT_SERVER_IP}"
                        SERVER_USER="ec2-user"
                        TOMCAT_DIR="/opt/tomcat/webapps"

                        echo "Copying WAR to Tomcat server..."

                        scp -o StrictHostKeyChecking=no \
                            "$WAR_FILE" \
                            "$SERVER_USER@$SERVER_IP:/tmp/"

                        echo "Moving WAR to Tomcat webapps..."

                        ssh -o StrictHostKeyChecking=no \
                            "$SERVER_USER@$SERVER_IP" \
                            "sudo mv /tmp/$(basename "$WAR_FILE") $TOMCAT_DIR/"

                        echo "Restarting Tomcat..."

                        ssh -o StrictHostKeyChecking=no \
                            "$SERVER_USER@$SERVER_IP" \
                            "sudo systemctl restart tomcat"

                        echo "Deployment completed successfully!"
                    '''
                }
            }
        }
    }
}
```
