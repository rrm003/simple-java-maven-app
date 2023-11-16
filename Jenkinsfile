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

        stage('Monitor with Stackdriver') {
            steps {
                script {
                    // Set up monitoring and logging with Stackdriver on GCP
                    sh "gcloud config set project ${GCP_PROJECT}"
                    sh "gcloud auth activate-service-account --key-file=${GOOGLE_CREDENTIALS_FILE}"
                    sh "gcloud components install stackdriver"
                    sh "gcloud beta auth application-default login"
                    sh "gcloud beta compute instances add-metadata ${GCE_INSTANCE_NAME} --zone=${GCP_ZONE} --metadata google-stackdriver-enabled=true"
                }
            }
        }
    }
}
