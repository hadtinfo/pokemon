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
                    sh returnStdout: true, script: './gradlew test'
                    RESULT_TEST = sh returnStdout: true, script: 'tail -n -5 test.log'
                    recordCoverage tools: [[parser: 'JACOCO', pattern: '**/jacoco/jacoco.xml']], sourceCodeRetention: 'EVERY_BUILD',
            qualityGates: [ [threshold: 90.0, metric: 'LINE', baseline: 'PROJECT', criticality: 'UNSTABLE']],
            sourceDirectories: [[path: 'plugin/src/main/java']]
                }
            }
        }
//        stage('jira') {
//            steps {
//                echo "zzzz"
//                echo "${env.RESULT_TEST}"
//                script {
//                    withCredentials([string(credentialsId: 'ghtk_jira_token', variable: 'GHTK_JIRA_TOKEN')]) {
//                        sh "curl --noproxy '*' --request POST  --url \'https://jira.ghtklab.com/rest/api/latest/issue/EW-4220/transitions\' --header \'Content-Type: application/json\' --header \'Authorization: Bearer ${GHTK_JIRA_TOKEN}\' --data \'{\" transition \":{\" id \":\" 171 \"}}\'"
//                    }
//                }
//            }
//        }
    }
    
  }
