pipeline { 
    agent any 
     
    tools { 
        nodejs 'node' 
    } 
     
    environment { 
        SCANNER_HOME = tool 'sonar-scanner' 
        AWS_ACCOUNT_ID="358308582535"
        REGION="ap-south-1"
    } 
 
    stages { 
        stage('Git Checkout') { 
            steps { 
                git credentialsId: 'kabeer-git-cred', url: 
'https://github.com/dev-kabeer04/3-Tier-Full-Stack-Application.git' 
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
                sh "trivy fs --format table -o fs-report.html ." 
            } 
        } 
         
        stage('SonarQube') { 
            steps { 
                withSonarQubeEnv('sonar') { 
                    sh "$SCANNER_HOME/bin/sonar-scanner 
Dsonar.projectKey=Campground -Dsonar.projectName=Campground" 
                } 
            } 
        } 
         
        stage('Docker Build & Tag') { 
            steps { 
                script { 
                    withDockerRegistry(credentialsId: 'kabeer-docker-cred', 
toolName: 'docker') { 
                        sh "docker build -t kabeer04/camp:latest ." 
                    } 
                } 
            } 
        } 
         
        stage('Trivy Image Scan') { 
            steps { 
                sh "trivy image --format table -o fs-report.html 
kabeer04/camp:latest" 
            } 
        } 
         
        stage('Docker Push Image') { 
            steps { 
                script { 
                    withDockerRegistry(credentialsId: 'kabeer-docker-cred', 
toolName: 'docker') { 
                        sh "docker push adijaiswal/campa:latest" 
                    } 
                } 
            } 
        } 

        stage('Deployment'){
            steps{
                script{
                    withAWS(credentials: 'aws-auth', region: "${REGION}") {
                        sh """
                        aws eks update-kubeconfig --region ${REGION} --name pixalive-cluster
                        cd Manifests
                        kubectl apply -f dss.yml
                        }
                        """
                    }
                }
            }
   }
}
}