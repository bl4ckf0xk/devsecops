pipeline {
    agent any

    stages {
        stage('Build Artifact - Maven') {
            steps {
                sh "mvn clean package -DskipTests=true"
                archive 'target/*.jar'
            }
        }

        stage('Unit Test') {
            steps {
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                } 
            }
        }

        stage('Docker build and push') {
            steps{
                withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                sh 'printenv'
                sh 'docker build -t bl4ckf0xk/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push bl4ckf0xk/numeric-app:""$GIT_COMMIT""'
            }
            }
        }
    }
}