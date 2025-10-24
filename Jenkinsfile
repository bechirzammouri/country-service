pipeline{
    agent any 
    tools {
        maven 'M2_HOME'
    }
    stages {
        stage('checkout code'){
            steps {
                git branch: 'main', url: 'https://github.com/bechirzammouri/country-service.git'
            }
        }

        stage ('compile , test code , package in jar file and store it in maven repository'){
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis'){
            steps {
                withSonarQubeEnv(installationName: 'MySonarQubeServer', credentialsId: 'sonarqube-PWD')  {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=country-service -Dsonar.projectName=country-service -Dsonar.host.url=http://localhost:9000 '
                }
            }
        }
    }
}