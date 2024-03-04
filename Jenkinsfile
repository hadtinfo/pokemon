pipeline {
    agent any
    environment {
        PROJECT = 'pokemon'
//    GIT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
        GIT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit").trim()
        GCHAT_CHANNEL_ID = '8742853339619397902'
    }

    stages {
        stage('test') {
            steps {
                script {
                    // inline
                    sh returnStdout: true, script: './gradlew test deltaCoverage'
                    recordCoverage(
                                    tools: [[parser: 'JACOCO'], pattern: '**/delta-coverage/report.xml']],
                                        id: 'jacoco', name: 'JaCoCo Coverage',
                                        sourceCodeRetention: 'EVERY_BUILD',
                                        qualityGates: [
                                                [threshold: 60.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
                                                [threshold: 60.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]]
                                )
                }
            }
        }
    }
    
  }
