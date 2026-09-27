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

        stage('Save WAR') {
            steps {
                sh '''
                    mkdir -p /home/ec2-user/warfiles
                    cp sample-app/target/*.war /home/ec2-user/warfiles/
                '''
            }
        }
    }
}

