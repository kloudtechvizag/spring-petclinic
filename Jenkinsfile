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
                   
                    // Path to the Maven-built JAR
                    env.JAR_FILE_PATH = "${env.WORKSPACE}/target/${artifactId}-${version}.jar"

                    def pom = new XmlSlurper().parse(new File("${env.WORKSPACE}/pom.xml"))
                    def groupId = pom.groupId.text() ?: pom.parent.groupId.text()
                    def artifactId = pom.artifactId.text()
                    def version = pom.version.text()
                    echo "GroupId: ${groupId}, ArtifactId: ${artifactId}, Version: ${version}"
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