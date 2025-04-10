pipeline {
    agent any

    environment {
        SONARQUBE = 'SonarQube'
        REGISTRY = 'ssethumadav/java-microservice'
    }

    tools {
        maven 'Maven'
        jdk 'jdk11'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: env.BRANCH_NAME, url: 'https://github.com/ssethumadav/devops-assesment.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            when {
                anyOf {
                    branch 'develop'
                    branch pattern: "feature/.*", comparator: "REGEXP"
                }
            }
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            when {
                branch 'develop'
            }
            steps {
                sh 'docker build -t $REGISTRY:$BUILD_NUMBER .'
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'develop'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push $REGISTRY:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'develop'
            }
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }
}
