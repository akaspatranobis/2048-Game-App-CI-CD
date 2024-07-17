pipeline {
    agent any 
    tools {
        jdk 'jdk17'
        nodejs 'nodejs'
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/akaspatranobis/2048-Game-App-CI-CD.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=2048-game \
                        -Dsonar.projectKey=2048-game '''
                    }
                
            }
        }
        stage('Quality Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        stage('Installing Dependencies') {
            steps {
                sh 'npm install'
                
            }
        }
        stage('OWASP Dependency-Check Scan') {
            steps {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                
            }
        }
        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
                
            }
        }
        stage('Ansible Docker') {
            steps {
                dir('Ansible'){
                  script {
                        ansiblePlaybook credentialsId: 'SSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml'
                    }     
                }    
            }
        }
        stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image apatranobis59/2048-game:latest > trivyimage.txt' 
            }
        }
        // stage('k8s using ansible'){
        //     steps{
        //         dir('Ansible') {
        //             script{
        //                 ansiblePlaybook credentialsId: 'SSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'kube.yaml'
        //             }
        //         } 
        //     }
        // }
    }
}
