pipeline { 
    agent any 
    tools{
        maven "maven"
    }
    environment {
        PATH = "$PATH:/usr/share/maven/bin"
    } 
    stages { 
        stage('Code Checkout') {
            steps {
                git url:"https://github.com/bhuvi-12/maven-project.git"
            }
        }
        stage('Build') { 
            steps { 
                sh 'mvn clean package'
            }
        }
        stage('Code analysis'){
            steps{
                withSonarQubeEnv('SonarQube-server') { 
                    sh "mvn sonar:sonar"
                }
            }
        }
        stage('Push artifacts from Dev branch'){
            when{
                branch 'dev'
            }
            steps{
                rtUpload(
                    serverId: 'jfrog-server',
                    spec: """{
                        "files": [
                                {
                                    "pattern": "/var/lib/jenkins/workspace/sample-java-project_dev/server/target/*.jar",
                                    "target": "libs-snapshot-local"
                                }
                        ]
                    }"""
                )
                rtUpload(
                    serverId: 'jfrog-server',
                    spec: """{
                        "files": [
                                {
                                    "pattern": "/var/lib/jenkins/workspace/sample-java-project_dev/webapp/target/*.war",
                                    "target": "libs-snapshot-local"
                                }
                        ]
                    }"""
                )
            }
        }
        stage('Push artifacts from Master branch'){
            when{
                branch 'master'
            }
            steps{
                rtUpload(
                    serverId: 'jfrog-server',
                    spec: """{
                        "files": [
                                {
                                    "pattern": "/var/lib/jenkins/workspace/sample-java-project_master/server/target/*.jar",
                                    "target": "libs-snapshot-local"
                                }
                        ]
                    }"""
                )
                rtUpload(
                    serverId: 'jfrog-server',
                    spec: """{
                        "files": [
                                {
                                    "pattern": "/var/lib/jenkins/workspace/sample-java-project_master/webapp/target/*.war",
                                    "target": "libs-snapshot-local"
                                }
                        ]
                    }"""
                )
            }
        }
        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: 'jfrog-server'
                )
            }
        }
        stage ('Deploy'){
            when{
                branch 'master'
            }
            steps{
                echo 'Development deployment'
                script {
                    deploy adapters: [tomcat9(credentialsId: 'tomcat-cred', path: '', url: 'http://tomcatserver.centralindia.cloudapp.azure.com:8080/')], contextPath: '', onFailure: false, war: 'webapp/target/*.war' 
                }
            }
        }
    }
}