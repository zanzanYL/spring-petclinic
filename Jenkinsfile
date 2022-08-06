pipeline {
    agent {
        docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    environment {
        SONAR_TOKEN = 'd000273d956d66f878c13535637bda8743ae51d6'
    }
    stages {
//         stage('Build') {
//             steps {
//                 sh 'echo $PATH; ls /usr/bin; mvn -B -DskipTests clean package'
//             }
//         }
//         stage('Test') {
//             steps {
//                 sh 'mvn test'
//                 step( [ $class: 'JacocoPublisher' ] )
//             }
//             post {
//                 always {
//                     junit 'target/surefire-reports/*.xml'
//                 }
//             }
//         }
//
//         stage('Sonarqube Analysis') {
//             steps {
//                 withSonarQubeEnv('SonarOne'){
//                    sh 'mvn sonar:sonar \
//                          -Dsonar.projectKey=zanzanYL_zanzanYL \
//                          -Dsonar.host.url=http://172.19.0.4:9000 \
//                          -Dsonar.login=d000273d956d66f878c13535637bda8743ae51d6 \
//                          -Dsonar.nodejs.executable=/usr/bin/node'
//
//                 }
//             }
//         }
//         stage("Quality gate") {
//             steps {
//                 waitForQualityGate abortPipeline: true
//             }
//         }
        stage('Deploy And Run App'){
            steps {
                script {
//                 ansiblePlaybook installation: 'ansible',
//                                 inventory: 'inventory', playbook: 'playbook.yml',
//                                 disableHostKeyChecking: true
                    sh 'ls -ltr /usr/local/bin | grep ansible; ls -ltr /usr/local/sbin | grep ansible; ls -ltr /usr/sbin | grep ansible; /usr/bin/ansible -i inventory playbook.yml'
                }
            }

        }
    }
    post {
        always {
            junit testResults: '**/target/surefire-reports/TEST-*.xml'
            step([
                $class           : 'JacocoPublisher',
                execPattern      : 'target/jacoco.exec',
                classPattern     : 'target/classes',
                sourcePattern    : 'src/main/java',
                exclusionPattern : 'src/test*'
            ])
            recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
        }
    }
}
