pipeline {
    agent {
        docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    environment {
        SONAR_TOKEN = '5ced6fabc0d7621798bf88144afe884b464217c7'
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

        stage ('CheckStyle Analysis') {
            steps {
                sh 'mvn --batch-mode -V -U -e checkstyle:checkstyle'
            }
        }

        stage ('Spotbugs Analysis') {
            steps {
                sh 'mvn --batch-mode -V -U -e com.github.spotbugs:spotbugs-maven-plugin:4.6.0.0:spotbugs'
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('SonarOne'){
                   sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=zanzanYL_zanzanYL'

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
            recordIssues enabledForFailure: true, tool: checkStyle(pattern: '**/target/checkstyle-result.xml')
            recordIssues enabledForFailure: true, tool: spotBugs(pattern: '**/target/spotbugsXml.xml')
        }
    }
}
