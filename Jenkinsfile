pipeline {
    agent {
        docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    environment {
        SONAR_TOKEN = 'c85de95f54d674b3e2ab4bd51b2b6b297b3df191'
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
                         -Dsonar.login=c85de95f54d674b3e2ab4bd51b2b6b297b3df191 \
                         -Dsonar.nodejs.executable=/usr/bin/node'

                }
            }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
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
