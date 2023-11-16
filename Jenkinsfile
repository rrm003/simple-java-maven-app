pipeline {
    agent any

    tools {
        maven 'maven' // Use the correct tool name 'maven'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Custom Maven settings if needed
                    def mavenHome = tool 'maven'
                    env.PATH = "${mavenHome}/bin:${env.PATH}"

                    // Run Maven build
                    sh 'mvn -B -DskipTests clean package'
                }
            }
        }
    }
}
