node {
    def app 
   
    stage('Clone repository') {
        /* Cloning the Repository to our Workspace */
            checkout scm

    }   

    stage('Linting') {
      
        docker.image('hadolint/hadolint:latest-debian').inside() {
                            sh 'hadolint ./Dockerfile | tee -a hadolint_lint.txt'
                            sh '''
                                lintErrors=$(stat --printf="%s"  hadolint_lint.txt)
                                if [ "$lintErrors" -gt "0" ]; then
                                    echo "Errors have been found, please see below"
                                    cat hadolint_lint.txt
                                    exit 1
                                else
                                    echo "There are no erros found on Dockerfile!!"
                                fi
                            '''
        }
        
    }
    stage('Build & Push to dockerhub') {
       
            app = docker.build("omaroovee/inmemorydb")
            docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                app.push("${env.BUILD_NUMBER}")
                app.push("latest")
            } 
        
    }

    stage('Deploying to EKS') {
        echo 'Deploying to AWS...'
        dir ('./') {
            withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                sh "aws eks --region us-east-1 update-kubeconfig --name udacity-capstone-EKS-Cluster"
                sh "kubectl apply -f aws/aws-auth-cm.yaml"
                sh "kubectl apply -f aws/capstone-app-deployment.yml"
                sh "kubectl get nodes"
                sh "kubectl get pods"
            }
        }
    }
    stage("Cleaning Docker up") {
      
        sh "echo 'Cleaning Docker up'"
        sh "docker system prune"
    
        
    }
    
}