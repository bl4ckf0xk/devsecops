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

        stage('Mutation Test - PIT'){
            steps{
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
                always {
                    pitmutation mutationCoverage: '**/target/pit-reports/**/mutation.xml'
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

        stage('Kubernetes Developement - Dev'){
            steps{
                withKubeConfig([credentialsId: 'kubeconfig']){
                    sh "sed -i 's#replace#bl4ckf0xk/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                    sh "kubectl apply -f k8s_deployment_service.yaml"
                }
            }
        }
    }
}