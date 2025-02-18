pipeline {
    agent any
    tools {
        maven 'maven' // Use the correct tool name 'maven'
    }
    environment {
        GOOGLE_CREDENTIALS = credentials('gcloud-creds')
        GCP_PROJECT = 'valut-svc'
        GCE_INSTANCE_NAME = 'instance-1'
        GCP_ZONE = 'us-central1-a'
        STORAGE_BUCKET = 'lab3artifacts'
        LOG_NAME = "jenkins-${JOB_NAME}-${BUILD_NUMBER}" 
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
        stage('Test') {
            steps {
                script {
                    // Run tests (replace with your testing commands)
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy to Google Cloud Storage') {
            steps {
                script {
                    // Copy the build output to Google Cloud Storage
                    withCredentials([file(credentialsId: 'gcloud-creds', variable: 'GOOGLE_CREDENTIALS_FILE')]) {
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_CREDENTIALS_FILE}"
                    }

                    // Replace 'target' with the actual build output directory
                    sh "gsutil cp -r target gs://${STORAGE_BUCKET}/"
                }
            }
        }
        stage('Download from Google Cloud Storage') {
            steps {
                script {
                    // Download the JAR file from Google Cloud Storage to the workspace
                    sh "gsutil cp gs://${STORAGE_BUCKET}/target/my-app-1.0-SNAPSHOT.jar ./"
                }
            }
        }
        stage('Deploy to Google Compute Engine') {
            steps {
                script {
                    // Authenticate with Google Cloud Platform using service account credentials
                    withCredentials([file(credentialsId: 'gcloud-creds', variable: 'GOOGLE_CREDENTIALS_FILE')]) {
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_CREDENTIALS_FILE}"
                    }

                    // Deploy to Google Compute Engine using gcloud commands
                    sh "gcloud config set project ${GCP_PROJECT}"
                    
                    // Copy the JAR file to the Compute Engine instance
                    sh "gcloud compute scp --zone=${GCP_ZONE} my-app-1.0-SNAPSHOT.jar ${GCE_INSTANCE_NAME}:~/"
                    // SSH into the instance and start the application
                    sh "gcloud compute ssh --zone=${GCP_ZONE} ${GCE_INSTANCE_NAME} --command 'nohup java -jar ~/my-app-1.0-SNAPSHOT.jar &'"
                }
            }
        }
    }
    post {
        always {
            script {
                // Send all logs to Google Cloud Logging
                withCredentials([file(credentialsId: 'gcloud-creds', variable: 'GOOGLE_CREDENTIALS_FILE')]) {
                    sh "gcloud auth activate-service-account --key-file=${GOOGLE_CREDENTIALS_FILE}"
                    sh "gcloud config set project ${GCP_PROJECT}"
                    sh "gcloud logging write ${LOG_NAME} '${JOB_NAME} build ${BUILD_NUMBER} completed'"
                }
            }
        }
    } 
}
