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

        stage('Deploy to Google Compute Engine') {
            steps {
                script {
                    // Authenticate with Google Cloud Platform using service account credentials
                    withCredentials([file(credentialsId: 'gcloud-creds', variable: 'GOOGLE_CREDENTIALS_FILE')]) {
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_CREDENTIALS_FILE}"
                    }

                    // Deploy to Google Compute Engine using gcloud commands
                    sh "gcloud config set project ${GCP_PROJECT}"
                    sh "gcloud compute ssh ${GCE_INSTANCE_NAME} --zone=${GCP_ZONE} --command='cd /path/to/your/application && ./deploy.sh'"
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
