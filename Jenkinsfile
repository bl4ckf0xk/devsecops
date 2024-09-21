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
        }

        stage('Mutation Test - PIT'){
            steps{
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
        }

        stage('SonarQube Analysis - SAST') {
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demo.southeastasia.cloudapp.azure.com:9000"
                }
                timeout(time: 2, unit: 'MINUTES') {
                    script {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Vulnerability Scan - Docker') {
            steps{
                sh "mvn dependency-check:check"
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

    post {
        always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
    }
}