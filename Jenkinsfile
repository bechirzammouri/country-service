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
                withSonarQubeEnv(installationName: 'MySonarQubeServer', credentialsId: 'sonarqube-token')  {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=country-service -Dsonar.projectName=country-service -Dsonar.host.url=http://172.17.96.1:9000 '
                }
            }
        }
        stage('Deploy to Nexus'){
            steps {
                sh 'mvn deploy -DskipTests -Dmaven.install.skip=true'
            }
        }
        
        stage('Download from Nexus and Deploy to Tomcat'){
            steps {
                sh '''
                    # Define variables
                    GROUP_ID=net.bechir
                    ARTIFACT_ID=service-country
                    VERSION=0.0.1-SNAPSHOT
                    PACKAGING=jar
                    NEXUS_URL=http://172.17.96.1:8081/repository/maven-snapshots
                    
                    # Download artifact from Nexus
                    curl -u admin:admin -o ${ARTIFACT_ID}.${PACKAGING} \
                    "${NEXUS_URL}/${GROUP_ID//.//}/${ARTIFACT_ID}/${VERSION}/${ARTIFACT_ID}-${VERSION}.${PACKAGING}"
                    
                    # Deploy to Tomcat Manager
                    curl -v -u deployer:deployer123 \
                    --upload-file ${ARTIFACT_ID}.${PACKAGING} \
                    "http://localhost:9090/manager/text/deploy?path=/country-service&update=true"
                '''
            }
        }
    }
}