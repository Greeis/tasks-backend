pipeline {
    agent any

    tools {
        jdk 'JAVA_LOCAL'
    }

    environment{
        scannerHome = tool 'SONAR_SCANNER'
        mavenHome = tool 'MAVEN_LOCAL'
        PATH = "$PATH:/usr/local/bin"
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
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=TestDeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=c002f2434f52a2faa29aa0b60b398e5e03463bdb -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/model/**,**Application.java,**/src/test/**"
                }
            }

        }
        stage('Quality Gate'){
            steps {
                sleep(10)
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
                dir('api-test'){
                    git branch: 'main', credentialsId: 'GitHubLogin', url: 'git@github.com:Greeis/tasks-api-tests.git'
                    sh "${mavenHome}/bin/mvn test"
                }
            }
        }
        stage('Deploy Front'){
            steps {
                dir('front-end'){
                    git branch: 'master', credentialsId: 'GitHubLogin', url: 'git@github.com:Greeis/tasks-frontend.git'
                    sh "${mavenHome}/bin/mvn clean package"
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage('Functional Tests'){
            steps {
                sleep(10)
                dir('funcional-test'){
                    git branch: 'master', credentialsId: 'GitHubLogin', url: 'git@github.com:Greeis/tasks-funcional-tests.git'
                    sh "${mavenHome}/bin/mvn -Dsurefire.rerunFailingTestsCount=2 test"
                }
            }
        }
        stage('Deploy Prod'){
            steps {
                sh '/usr/local/bin/docker-compose build'
                sh '/usr/local/bin/docker-compose up -d'
            }
        }
        stage('Health Check'){
            steps {
                sleep(10)
                dir('funcional-test'){
                    sh 'mvn verify -Dskip.surefire.tests -Dsurefire.rerunFailingTestsCount=2'
                }
            }
        }     
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, funcional-test/target/surefire-reports/*.xml, funcional-test/target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, front-end/target/tasks.war', followSymlinks: false, onlyIfSuccessful: true
        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build $BUILD_NUMBER has failed', to: 'graziele_182@hotmail.com'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build is fine!!', to: 'graziele_182@hotmail.com'
        }
    }
}

