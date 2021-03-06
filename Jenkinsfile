pipeline {
    agent {
        docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    tools {
        nodejs 'nodejs'
    }
    environment {
        SONAR_TOKEN = 'd000273d956d66f878c13535637bda8743ae51d6'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
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
                         -Dsonar.login=d000273d956d66f878c13535637bda8743ae51d6 \
                         -Dsonar.nodejs.executable=/usr/bin/node'

                }
            }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        stage('Deploy Run Petclinic'){
            steps {
                script {
                    ansiblePlaybook installation: 'ansible2',
                    inventory: 'host', playbook: 'ansible.yml',
                    disableHostKeyChecking: true
                    // check below code for IP ssh based deployment
                    // for different Ips
                    // IP address and role goes in dev.inv
                    /**[webservers]
                    IP-address ansible_user=ec2-user
                    **/
                    // command changes to include crendeitalsId
                    // private-key values if your jenkins configured key to connect to server IP
                    // check the screenshot you have
                    // ansiblePlaybook crendeitalsId: 'private-key',
                    // installation: 'ansible2',
                    // inventory: 'hosts',
                    // playbook: 'ansible.yml',
                    // disableHostKeyChecking: true
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
