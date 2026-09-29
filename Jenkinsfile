
pipeline {
    agent any

    parameters { 
        choice(
            name: 'TOMCAT_SERVER',
            choices: [ 
                'Tomcat QA (13.207.198.17)', 
                'Test3 tomcat (13.206.108.184)'
            ],
                 description: 'Select Tomcat server' )
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

                        SERVER_IP=$(echo "$TOMCAT_SERVER" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+')

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
