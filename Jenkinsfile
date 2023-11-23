pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "JDK11"
    }
    stages{
        stage('Fetch Code from gitHub'){
            steps{
                git branch: 'master', url: 'https://github.com/starlightng/maven-web-application.git'
            }
        }

        stage('Build'){
            steps{
                sh 'mvn install -DskipTests'
            }
            post{
                success{
                    echo "Archiving the Artifacts"
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('Checkstyle Analysis'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar: Analysing the code and uploading result to SonarServer'){
            environment{
                scannerHome = tool 'sonar4.7'      
            }
            steps{
                withSonarQubeEnv('sonar'){
                    sh '''${scannerHome}/bin/sonar-scanner \
                     -Dsonar.projectKey=maven_webapp \
                     -Dsonar.projectName=maven_webapp \
                     -Dsonar.projectVersion=1.0 \
                     -Dsonar.sources=src/ \
                     -Dsonar.java.binaries=target/classes/com/mt/services/ \
                     -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                     '''
                }
            }
        }

        stage('UploadArtifacts to NexusServer'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.31.51.159:8081',
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: 'Learn_withPride-Repo',
                    credentialsId: 'NexusLogin',
                    artifacts: [
                        [artifactId: 'maven-app',
                        classifier: '',
                        file: 'target/maven-web-app.war',
                        type: 'war']
                    ]
                )
            }
        }

        stage('Deploy to Tomcat'){
            steps{
                deploy adapters: [tomcat9(credentialsId: 'tomcatCred',
                                  path: '',
                                  url: 'http://172.31.38.124:8080/' )],
                                  contextPath: null,
                                  war: 'target/*.war'
            }
        }
    }
}
