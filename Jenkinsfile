
pipeline {
    agent any

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

        stage('Save WAR') {
            steps {
                sh '''
                    cp sample-app/target/*.war /home/ec2-user/warfiles/
                '''
            }
        }
    }
}


