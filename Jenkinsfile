pipeline {
    agent any 
    stages{

        stage("BuildCode"){
            steps {
                sh """
                    ./mvnw package -DskipTests
                    ls -lrt target/*.jar

                """
            }
        }

        // stage("Run-Unit-Tests"){
        //     steps {
        //         sh """
        //             ./mvnw test

        //         """
        //     }
        // }

        stage("Sonar-Scanning"){
            steps {
               
               script {
                    withSonarQubeEnv( installationName: 'dev-sonar-01', credentialsId: 'jenkins-sonar-creds') {
                        sh """
                            sonar-scanner \
                                        -Dsonar.projectKey=petclinic \
                                        -Dsonar.projectName=petclinic \
                                        -Dsonar.exclusions=**/*.java \
                                        -Dsonar.sources=. \
                                        -Dsonar.java.binaries=target/classes \
                                        -Dsonar.sourceEncoding=UTF-8
                        """
                    }
               }
            }
        }

        stage("Sonar Quality Gate") {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline failed due to quality gate: ${qg.status}"
                        }
                    }
                }
            }
        }


        stage("Upload-Artifacts-Nexus") {
            steps {
                script {
                    // Load Maven coordinates dynamically from pom.xml
                    def pom = readMavenPom file: 'pom.xml'
                    def groupId = pom.groupId ?: pom.parent.groupId
                    def artifactId = pom.artifactId
                    def version = pom.version

                    // Optionally append Jenkins build number for snapshot version
                    def snapshotVersion = version.replace('-SNAPSHOT', "-${env.BUILD_NUMBER}-SNAPSHOT")

                    // Path to the Maven-built JAR
                    env.JAR_FILE_PATH = "${env.WORKSPACE}/target/${artifactId}-${version}.jar"

                    echo "Uploading artifact:"
                    echo "GroupId: ${groupId}"
                    echo "ArtifactId: ${artifactId}"
                    echo "Version: ${snapshotVersion}"
                    echo "JAR Path: ${env.JAR_FILE_PATH}"

                    // Upload to Nexus
                    // nexusArtifactUploader(
                    //     nexusVersion: 'nexus3',
                    //     protocol: 'http',
                    //     nexusUrl: 'nexus:8081',
                    //     groupId: groupId,
                    //     version: snapshotVersion,
                    //     repository: 'maven-snapshots',
                    //     credentialsId: 'nexus-creds',
                    //     artifacts: [
                    //         [
                    //             artifactId: artifactId,
                    //             file: env.JAR_FILE_PATH,
                    //             type: 'jar',
                    //             classifier: ''
                    //         ]
                    //     ]
                    // )
                }
            }
        }

        // stage("Deploy-Dev"){
        //     steps {
        //         sshagent(['jenkins-aws-ssh-creds']) {
        //             sh """
        //                 scp -o StrictHostKeyChecking=no ${env.JAR_FILE_PATH} ubuntu@65.0.251.218:/home/ubuntu
        //             """
        //         }
        //     }
        // }

        // stage("Deploy-UAT"){
        //     steps {
        //         sshagent(['jenkins-aws-ssh-creds']) {
        //             sh """
        //                 scp -o StrictHostKeyChecking=no ${env.JAR_FILE_PATH} ubuntu@43.204.211.49:/home/ubuntu
        //             """
        //         }
        //     }
        // }

        // stage("Deploy-PROD"){
        //     steps {
        //         sshagent(['jenkins-aws-ssh-creds']) {
        //             sh """
        //                 scp -o StrictHostKeyChecking=no ${env.JAR_FILE_PATH} ubuntu@13.200.235.238:/home/ubuntu
        //             """
        //         }
        //     }
        // }





    }
}