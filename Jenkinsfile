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
        
        stage('Download Artifact from Nexus'){
            steps {
                sh '''
                    # Define variables
                    GROUP_ID=net.bechir
                    ARTIFACT_ID=service-country
                    VERSION=0.0.1-SNAPSHOT
                    PACKAGING=jar
                    NEXUS_URL=http://172.17.96.1:8081/repository/maven-snapshots
                    
                    # Download artifact from Nexus
                    echo "Downloading ${ARTIFACT_ID}-${VERSION}.${PACKAGING} from Nexus..."
                    curl -u admin:admin -o ${ARTIFACT_ID}.${PACKAGING} \
                    "${NEXUS_URL}/${GROUP_ID//.//}/${ARTIFACT_ID}/${VERSION}/${ARTIFACT_ID}-${VERSION}.${PACKAGING}"
                    
                    # Verify download
                    ls -lh ${ARTIFACT_ID}.${PACKAGING}
                '''
            }
        }
        
        stage('Deploy to Tomcat'){
            steps {
                sh '''
                    # Deploy to Tomcat Manager
                    echo "Deploying service-country.jar to Tomcat..."
                    curl -v -u deployer:deployer123 \
                    --upload-file service-country.jar \
                    "http://localhost:9090/manager/text/deploy?path=/country-service&update=true"
                '''
            }
        }
    }
}