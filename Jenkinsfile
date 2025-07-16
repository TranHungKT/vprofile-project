pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
		NEXUS_USER = 'admin'
		NEXUS_PASS = 'admin'
		RELEASE_REPO = 'vprofile-release'
		CENTRAL_REPO = 'vpro-maven-central'
		NEXUSIP = '3.107.70.98'
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        registryCredential = 'ecr:ap-southeast-2:awscreds'
        appRegistry = '407558439482.dkr.ecr.ap-southeast-2.amazonaws.com/vprofileappimg'
        vprofileRegistry = 'https://407558439482.dkr.ecr.ap-southeast-2.amazonaws.com' 
        cluster = 'vprofileapptask-service-2kgymgdx'
        service = 'vprofileapptask-service-2kgymgdx'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }

            post {
                success {
                    echo "Now Archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage("Build App Image"){
            steps {
                script { 
                    dockerImage = docker.build(appRegistry + ":$BUILD_NUMBER" , "./Docker-files/app/multistage")
                }
            }
        }

        stage("Update App Image") {
            steps {
                script {
                    docker.withRegistry(vprofileRegistry, registryCredential) {
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage("Deploy to aws") {
            steps {
                withAWS(credentials: 'awscreds', region: 'ap-southeast-2') {
                    sh 'aws ecs update-sevice --cluster ${cluster} --service ${service} --force-new-deployment'
                }
            }
        }
    }
}