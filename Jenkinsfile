pipeline {
    agent any

    environment {
        registry = "innovativeacademy/academyapp"
        registryCredentials = 'Dockerhub'
        dbHostname = "innovative.cr3y72ybxjad.ap-south-1.rds.amazonaws.com"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Modify Properties File') {
            steps {
                script {
                    // Use sed to replace or add properties in application.properties
                    sh "sed -i 's|innovative.cuklu9atnt3c.ap-south-1.rds.amazonaws.com|${dbHostname}|g' src/main/resources/application.properties"

                    // Check if the property exists and add it if it doesn't
                    sh "grep -q '${dbHostname}' src/main/resources/application.properties || echo 'new_property=${dbHostname}' >> src/main/resources/application.properties"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving..."
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('Build App Image') {
            steps {
                script {
                    dockerImage = docker.build "${registry}:Innovative${BUILD_ID}"
                }
            }
        }

        stage('Upload Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredentials) {
                        dockerImage.push("Innovative${BUILD_ID}")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Kubernetes Deploy') {
            agent { label 'KOPS' }
            steps {
                sh "helm upgrade --install --force vprofile-stack helm/innovativecharts --set appimage=${registry}:Innovative${BUILD_ID} --namespace prod"
            }
        }
    }
}
