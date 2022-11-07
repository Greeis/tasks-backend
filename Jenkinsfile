pipeline {
    agent any

    tools {
        jdk 'JAVA_LOCAL'
    }

    environment{
        scannerHome = tool 'SONAR_SCANNER'
        mavenHome = tool 'MAVEN_LOCAL'
    }
    stages{

        stage ('Build') {
            steps{
                sh "${mavenHome}/bin/mvn clean package -DskipTests=true"
            }
        }
        stage ('Unit Tests') {
            steps {
                sh "${mavenHome}/bin/mvn test"
            }
        }
        stage('Sonar Analysis'){
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=TestDeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=6e8fca50937b291c2b950223d5e4b36754140fa8 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/model/**,**Application.java,**/src/test/**"
                }
            }

        }
        stage('Quality Gate'){
            steps {
                sleep(5)
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy Backend'){
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage('API Tests'){
            steps {
                git branch: 'main', credentialsId: 'GitHubLogin', url: 'git@github.com:Greeis/tasks-api-tests.git'
                sh "${mavenHome}/bin/mvn test"
            }
        }      
    }
}