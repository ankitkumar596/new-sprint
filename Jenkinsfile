def deployScript = {
    // Copy the JAR to the remote server using SCP
    sh "scp target/spring-boot-tutorial-0.0.1-SNAPSHOT.jar ubuntu@18.234.219.114:/home/ubuntu/${params.ENVIRONMENT.toLowerCase()}"

    // SSH into the server and kill the current process
    sh 'ssh -tt ubuntu@18.234.219.114 "sudo pkill -f spring-boot-tutorial-0.0.1-SNAPSHOT.jar"'
}

pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps {
                cleanWs()
                checkout scm: [
                    $class: 'GitSCM', 
                    branches: [
                        [name: '*/master']
                    ], 
                    userRemoteConfigs: [
                        [credentialsId: 'github_token', url: 'https://github.com/ankitkumar-devops/spring-boot-tutorial.git']
                    ]
                ]
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def mvn = tool 'MAVEN_HOME';
                    withSonarQubeEnv() {
                        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=spring-boot-tutorial -Dsonar.projectName='spring-boot-tutorial'"
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            when {
                expression {
                    return params.ENVIRONMENT == 'DEV'
                }
            }
            steps {
                script {
                    deployScript()
                }
            }
        }

        stage('Deploy to QA') {
            when {
                expression {
                    return params.ENVIRONMENT == 'QA'
                }
            }
            steps {
                script {
                    deployScript()
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                expression {
                    return params.ENVIRONMENT == 'PROD'
                }
            }
            steps {
                script {
                    deployScript()
                }
            }
        }

        stage('Deploy to All') {
            when {
                expression {
                    return params.ENVIRONMENT == 'ALL'
                }
            }
            steps {
                script {
                    def environments = ['DEV', 'QA', 'PROD']
                    for (def env in environments) {
                        echo "Deploying to ${env} environment"
                        deployScript()
                    }
                }
            }
        }
    }
}
