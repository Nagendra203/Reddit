pipeline{
    agent any
    tools{
        jdk "jdk17"
        nodejs "node16"
    }
    environment{
        SCANNER_HOME=tool "sonar-scanner"
    }
    stages{
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('checkout scm'){
            steps{
                git 'https://github.com/Nagendra203/Reddit.git'
            }
        }
        stage('install dependencies'){
            steps{
                sh 'npm install'
            }
        }
        stage('sonarqube analysis'){
            steps{
                script{
                    withSonarQubeEnv('sonar-server') {
                       sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit \
                       -Dsonar.projectKey=reddit'''
                    }
                }
            }
        }
        stage('trivy file scan'){
            steps{
                sh 'trivy fs .'
            }
        }
        stage('owasp dependency check'){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build & Push Image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker') {
                        sh 'docker build -t reddit .'
                        sh 'docker tag reddit nagendra206/reddit:latest'
                        sh 'docker push nagendra206/reddit:latest'
                    }
                }
            }
        }
        stage('trivy image scan'){
            steps{
                sh 'trivy image nagendra206/reddit:latest'
            }
        }
        stage('Deploy'){
            steps{
                sh 'docker run -d -p 3000:3000 --name redditapp nagendra206/reddit:latest'
            }
        }
        stage(k8s){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh 'kubectl apply -f deployment.yml' 
                    }
                }
            }
        }
    }
    post {
        always {
            emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                    "Build Number: ${env.BUILD_NUMBER}<br/>" +
                    "URL: ${env.BUILD_URL}<br/>",
            to: 'dammatinagendrababu.marolix@gmail.com',
            attachmentsPattern: 'trivy.txt'
        }
    }
}