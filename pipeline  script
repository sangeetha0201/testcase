pipeline {
    agent none
    stages {
        stage('SonarQube analysis') {
            agent any
            steps {
                withSonarQubeEnv('local-sonar1') {
                   bat 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install org.jacoco:jacoco-maven-plugin:report'
                   bat 'mvn sonar:sonar' 
                }
            }
        }
        stage("Quality Gate") {
            agent any
            steps {
                sleep(10)
                timeout(time: 5, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('maven build'){
            agent {
                label 'ubuntu-slave-1'
              }
            steps{
                sh 'mvn clean install'
                sh 'mv ${WORKSPACE}/target/*.war ${WORKSPACE}/target/hello-spring-boot-war-${BUILD_NUMBER}.war'
            }
        }
        stage('upload war to s3'){    
            agent {
                label 'ubuntu-slave-1'
              }       
            steps{
                s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'sangeetha-jenkins-war', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: false, selectedRegion: 'ap-south-1', showDirectlyInBrowser: false, sourceFile: 'target/hello-spring-boot-war-${BUILD_NUMBER}.war', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'FAILURE', profileName: 's3-uploads', userMetadata: []
            }
        }
        stage('deploy to tomcatserver'){   
            agent {
                label 'ubuntu-slave-1'
              }        
            steps{
                deploy adapters: [tomcat9(credentialsId: 'tomcat-creds', path: '', url: 'http://3.108.67.15:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        }
    }
