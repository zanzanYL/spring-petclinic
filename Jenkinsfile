pipeline {
    agent any
//     {
//         docker {
//             image 'maven:3.8.1-adoptopenjdk-11'
//             args '-v /root/.m2:/root/.m2'
//         }
//     }
    environment {
        SONAR_TOKEN = 'd000273d956d66f878c13535637bda8743ae51d6'
        PATH = "/usr/bin/:$PATH"
        ANS_HOME = tool('ansible')
    }
    stages {
        stage('List') {
            steps {
                sh 'echo $PATH; ls -l /usr/bin;'
            }
        }
        stage('Build') {
            steps {
                sh 'echo $PATH; ls /usr/bin; mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
                step( [ $class: 'JacocoPublisher' ] )
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('SonarOne'){
                   sh 'mvn sonar:sonar \
                         -Dsonar.projectKey=zanzanYL_zanzanYL \
                         -Dsonar.host.url=http://172.19.0.2:9000 \
                         -Dsonar.login=4cc57a11c752288a2746a31b5f4688aa8b78fefb \
                         -Dsonar.nodejs.executable=/usr/bin/node'

                }
            }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        stage('Deploy And Run App'){
            steps {
                script {
                ansiblePlaybook installation: 'ansible',
                                inventory: 'inventory', playbook: 'playbook.yml',
                                disableHostKeyChecking: true
//                     echo "PATH is: $ANS_HOME"
//                     sh "ls -ltr /usr/local/bin; pwd; hostname -I; ls -ltr /var/jenkins_home/workspace/spring-petclinic/; /var/jenkins_home/workspace/spring-petclinic/ansible -i inventory playbook.yml"
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
