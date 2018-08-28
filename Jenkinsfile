pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'deploy_jenkins', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]){
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers:[
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo systemctl list-unit-files | grep enabled | grep train; if [ $? -eq "0" ]; then sudo /usr/bin/systemctl stop train-schedule; fi && sudo rm -rf /opt/train-schedule/* && sudo unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'   
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment looks good?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'deploy_jenkins', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]){
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers:[
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo systemctl list-unit-files | grep enabled | grep train; if [ $? -eq "0" ]; then sudo /usr/bin/systemctl stop train-schedule; fi && sudo rm -rf /opt/train-schedule/* && sudo unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'   
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }        
    }
}
