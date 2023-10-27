pipeline {
    agent any
    tools {
        maven 'Maven'
        nodejs 'Nodejs'
    }
    stages {
        stage('Source') {
            steps {
                git branch: 'master', url: 'https://github.com/Ahmedgit7/jenkins-training-CI-CD-java.git'
            }
        }

        stage('Build and SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    withMaven(maven: 'Maven') {
                        sh 'mvn clean verify sonar:sonar -Dsonar.javaOpts="-Xmx2g"'
                    }
                }
            }
        }

        stage('Publish to Nexus Repository Manager') {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: '54.67.18.31:8081/', // Corrected the URL
                            groupId: "${pom.groupId}",
                            version: "${pom.version}",
                            repository: 'Ahmed',
                            credentialsId: 'nexus',
                            artifacts: [
                                [
                                    artifactId: "${pom.artifactId}",
                                    classifier: '',
                                    file: artifactPath,
                                    type: "${pom.packaging}"
                                ],
                                [
                                    artifactId: "${pom.artifactId}",
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"
                                ]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }

        stage('Pull Artifact from Nexus') {
            steps {
                script {
                    def nexusRepoUrl = 'http://54.67.18.31:8081/repository/Ahmed/'
                    def groupId = 'com.mt' // Corrected the group ID
                    def artifactId = 'java-web-app' // Corrected the artifact ID
                    def version = '1.0' // Corrected the version
                    def packaging = 'war'

                    // Download the artifact
                    sh "curl -O ${nexusRepoUrl}${groupId}/${artifactId}/${version}/${artifactId}-${version}.${packaging}"
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def artifactId = 'java-web-app'
                    def version = '1.0'
                    def packaging = 'war'
                    def tomcatWebappsDir = '/usr/local/tomcat9/webapps'

                    // Copy the downloaded artifact to the Tomcat webapps directory
                    sh "cp ${artifactId}-${version}.${packaging} ${tomcatWebappsDir}/"
                }
            }
        }
    }
}
