pipeline {
    agent any
    tools {
        nodejs 'node'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        SONAR_TOKEN = credentials('sonar-token')
        AWS_ACCOUNT_ID = "358308582535"
        REGION = "ap-south-1"
    }
    stages {
        stage('Git Checkout') {
            steps {
                git credentialsId: 'kabeer-git-cred', branch: 'main', url: 'https://github.com/dev-kabeer04/3-Tier-Full-Stack-Application.git'
            }
        }
        stage('Install Package Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Unit Tests') {
            steps {
               sh "npm test"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o /tmp/trivy/report.html ."
            }
        }
        stage('OWASP Dependency-Check') {
             steps {
        // sh "dependency-check.sh --scan . --format HTML --project camp --out owasp-report.html"
                dependencyCheck additionalArguments: '--scan ./ --disableNpmAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.login=$SONAR_TOKEN -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"
                }
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'kabeer-docker-cred', toolName: 'docker') {
                        sh "docker build -t kabeer04/camp:latest ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html kabeer04/camp:latest"
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'kabeer-docker-cred', toolName: 'docker') {
                        sh "docker push kabeer04/camp:latest"
                    }
                }
            }
        }
        stage('Deployment'){
            steps{
                script{
                    withAWS(credentials: 'aws-credentials', region: "${REGION}") {
                        sh """
                        aws eks update-kubeconfig --region ${REGION} --name pixalive-cluster
                        cd helm
                        helm upgrade yelp-camp .
                        """
                    }
                }
            }
        }
   }
}