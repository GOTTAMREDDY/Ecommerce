pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'projects/ecommerce'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'Git_cred', url: 'https://github.com/GOTTAMREDDY/Ecommerce.git'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Maven Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }

        //stage('File System Scan') {
        //    steps {
         //       sh 'trivy fs --format table -o trivy-fs-report.html .'
         //   }
        //}

        //stage('SonarQube Analysis') {
        //    steps {
         //       withSonarQubeEnv('sonar') {
         //           sh '''
        //            $SCANNER_HOME/bin/sonar-scanner \
        //            -Dsonar.projectName=ECommerce \
        //            -Dsonar.projectKey=ECommerce \
        //            -Dsonar.java.binaries=target/classes
        //            '''
        //        }
        //    }
        //}

        stage('Maven Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 'maven') {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
                    sh "docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} 173640965114.dkr.ecr.us-east-1.amazonaws.com/projects/ecommerce:${BUILD_NUMBER}"
                }
            }
        }
        stage('ECR login') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 173640965114.dkr.ecr.us-east-1.amazonaws.com'
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                archiveArtifacts artifacts: 'trivy-image-report.html', fingerprint: true
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push 173640965114.dkr.ecr.us-east-1.amazonaws.com/projects/ecommerce:${BUILD_NUMBER}"
                }
            }
        }

        // stage('Deploy to Container') {
        //     steps {
        //         script {
        //             sh '''
        //     docker stop ecommerce-container || true
        //     docker rm ecommerce-container || true
        //     '''

        //             sh """
        //     docker run -d \
        //     --name ecommerce-container \
        //     -p 8083:8080 \
        //     ${DOCKER_IMAGE}:${BUILD_NUMBER}
        //     """
        //         }
        //     }
        // }
    stage('Deploy to Cluster') {
            steps {
                script {
                    sh "kubectlctl apply -f deployment-service.yaml "
                }
            }
        }
    }

}
