pipeline {
    agent {
        label "master"
    }
    tools {
        maven "Maven"
    }
    environment {
        //Nexus
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.250.12.89:8081"
        NEXUS_REPOSITORY = "maven-nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
        // DockerHub
        registry = "e.g. lylio/chatty"
        registryCredential = 'dockerhub_id'
        dockerImage = ''
    }
    stages {
        stage("Clone code from VCS") {
            steps {
                script {
                    git 'https://github.com/Lylio/chatty-services.git';
                }
            }
        }

    stage('Docker Build') {
        steps {
            script {

                dockerImage = docker.build registry + ":$BUILD_NUMBER"

                        }
                    }
                }

                stage('Deploy our image') {

                            steps {

                                script {

                                    docker.withRegistry( '', registryCredential ) {

                                        dockerImage.push()

                                    }

                                }

                            }

                        }

                        stage('Cleaning up') {

                            steps {

                                sh "docker rmi $registry:$BUILD_NUMBER"

                            }

                        }

        stage("Maven Build") {
            steps {
                script {
                    sh "mvn package -DskipTests=true"
                }
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
}