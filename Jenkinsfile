pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    tools {
        maven 'maven3'
        jdk 'jdk-8'
    }

    stages {

        stage('git checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/rameshwarsangewar/Ekart.git'
            }
        }

        stage('compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('unit tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube analysis') {
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
                withCredentials([
                    string(
                        credentialsId: 'nvd-api-key',
                        variable: 'NVD_API_KEY'
                    )
                ]) {
                    dependencyCheck(
                        additionalArguments: "--nvdApiKey=${NVD_API_KEY}",
                        odcInstallation: 'DC'
                    )
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('deploy to Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'global-maven',
                    jdk: 'jdk-8',
                    maven: 'maven3',
                    mavenSettingsConfig: '',
                    traceability: true
                ) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('build and Tag docker image') {
            steps {
                sh '''
                    docker build \
                    -t youngminds73/ekart:latest \
                    -f docker/Dockerfile .
                '''
            }
        }

        stage('Push image to Hub') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'dockerhub-pwd',
                        variable: 'DOCKERHUB_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_PASSWORD" | docker login \
                        -u youngminds73 \
                        --password-stdin

                        docker push youngminds73/ekart:latest
                    '''
                }
            }
        }

        stage('EKS and Kubectl configuration') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                    --region ap-south-1 \
                    --name project-cluster
                '''
            }
        }

        stage('Deploy to k8s') {
            steps {
                sh '''
                    kubectl apply -f deploymentservice.yml
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the console output for details.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
