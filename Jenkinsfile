pipeline {
    agent {
        docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    environment {
        SONAR_TOKEN = '76c2c7e545f481eea10dbe4eaad10eea3ffac33a'
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
                         -Dsonar.host.url=http://172.19.0.4:9000 \
                         -Dsonar.login=76c2c7e545f481eea10dbe4eaad10eea3ffac33a \
                         -Dsonar.nodejs.executable=/usr/bin/nodejs'

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
