pipeline{
    agent any
    tools{
        nodejs 'node21'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage('git checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/mukeshr-29/project-3tier-application-map.git'
            }
        }
        stage('install dependencies'){
            steps{
                sh 'npm install'
            }
        }
        stage('unit testing'){
            steps{
                sh 'npm test'
            }
        }
        stage('trivy filesystem scan'){
            steps{
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        stage('static code analysis'){
            steps{
                script{
                    withSonarQubeEnv('sonar-server'){
                        sh '$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=3-tier-prod -Dsonar.projectName=3-tier-prod'
                    }
                }
            }
        }
        stage('quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('docker build and tag'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                        sh 'docker build -t mukeshr29/3-tier-camp:prod .'
                    }
                }
            }
        }
        stage('docker img scan'){
            steps{
                sh 'trivy image --format table -o img.report mukeshr29/3-tier-camp:prod'
            }
        }
        stage('docker img push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                        sh 'docker push mukeshr29/3-tier-camp:prod'
                    }
                }
            }
        }
        stage('k8s deployment'){
            steps{
                script{
                    dir('Manifests'){
                        withKubeConfig(caCertificate: '', clusterName: '3-tier', contextName: '', credentialsId: 'k8s', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://0D17076E0B2094750553F6E55C0BEEEF.gr7.us-east-1.eks.amazonaws.com') {
                            sh 'kubectl apply -f dss.yml'
                            sleep 60
                        }
                    }
                }
            }
        }
        stage('verify deployment'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '3-tier', contextName: '', credentialsId: 'k8s', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://0D17076E0B2094750553F6E55C0BEEEF.gr7.us-east-1.eks.amazonaws.com') {
                        sh 'kubectl get pods'
                        sh 'kubectl get svc'
                    }
                }
            }
        }
    }
}