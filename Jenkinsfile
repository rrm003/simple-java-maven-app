pipeline {
    agent any

    tools {
        maven 'Maven 3.9.5' // Make sure this tool name matches the Maven tool configured in Jenkins
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Custom Maven settings if needed
                    def mavenHome = tool 'Maven 3.9.5'
                    env.PATH = "${mavenHome}/bin:${env.PATH}"

                    // Run Maven build
                    sh 'mvn -B -DskipTests clean package'
                }
            }
        }
    }
}
