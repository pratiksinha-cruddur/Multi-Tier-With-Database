pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/pratiksinha-cruddur/Multi-Tier-With-Database.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        
        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --format table -o file-system-scan.html .'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar'){
                	sh '$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Multitier -Dsonar.projectKey=Multitier -Dsonar.java.binaries=target' 
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        /*
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        */
        
        stage('Docker Build Image and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t pratiksinha20/bankapp:latest .'
                    }
                }
            }
        }
        
        stage('Docker Image Scan Using Trivy') {
            steps {
                sh 'trivy image --format table -o docker-image-scan.html pratiksinha20/bankapp:latest'
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push pratiksinha20/bankapp:latest '
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' eks-cluster', contextName: '', credentialsId: 'kubernetes-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://55011D00E377047A482B4135B5EDBABA.gr7.ap-south-2.eks.amazonaws.com') {
                    sh 'kubectl apply -f ds.yml'
                    sleep 30
                }
            }
        }
        
        stage('Verify Kubernetes Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' eks-cluster', contextName: '', credentialsId: 'kubernetes-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://55011D00E377047A482B4135B5EDBABA.gr7.ap-south-2.eks.amazonaws.com') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
        
    }
}
