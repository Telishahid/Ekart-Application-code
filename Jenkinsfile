pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        NVD_API_KEY = credentials('nvd-api-key')  // Jenkins secret text credential
    }

    tools {
        maven 'maven'
        jdk 'jdk'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Telishahid/Ekart-Application-code.git'
            }
        }

        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh "${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                  withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--nvdApiKey=$NVD_API_KEY",
                                    odcInstallation: 'DC'
             }
        }
        }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        

        stage('build and Tag docker image') {
            steps {
                script {
                        sh "docker build -t shahidteli/ekart:latest -f docker/Dockerfile ."
                    }
            }
        }

        stage('Push image to Hub'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                   sh 'docker login -u shahidteli -p ${dockerhubpwd}'}
                   sh 'docker push shahidteli/ekart:latest'
                }
            }
        }
        stage('EKS and Kubectl configuration'){
            steps{
                script{
                    sh 'aws eks update-kubeconfig --region us-west-2 --name shahid-cluster'
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                script{
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }
    }

}
