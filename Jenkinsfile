pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/rameshwarsangewar/Ekart.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    additionalArguments: '--scan . --format HTML --out dependency-check-report',
                    odcInstallation: 'DC'
                )
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'global-maven',
                    jdk: 'jdk-17',
                    maven: 'maven3',
                    mavenSettingsConfig: '',
                    traceability: true
                ) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                sh 'docker build -t admin1ramu/ekart:latest -f docker/Dockerfile .'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'dockerhub-pwd',
                        variable: 'dockerhubpwd'
                    )
                ]) {
                    sh '''
                        echo "$dockerhubpwd" | docker login \
                        -u admin1ramu \
                        --password-stdin

                        docker push admin1ramu/ekart:latest
                    '''
                }
            }
        }

        stage('EKS and Kubectl Configuration') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                    --region ap-south-1 \
                    --name project-cluster
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deploymentservice.yml'
            }
        }
    }
}
