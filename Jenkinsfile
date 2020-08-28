pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTest=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=24e1e4ff918a6d400cbf90e43efe378e3d509c16 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(20)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true 
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Tests') {
            steps {
                dir('api-tests'){
                    git credentialsId: 'github_login', url: 'https://github.com/pauloabsouto/api-tests'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('frontend'){
                    git credentialsId: 'github_login', url: 'https://github.com/pauloabsouto/tasks-frontend'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
                
            }
        }
        stage ('e2e Tests') {
            steps {
                dir('e2e-tests'){
                    git credentialsId: 'github_login', url: 'https://github.com/pauloabsouto/e2e-tests'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Prod'){
            steps {
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        stage ('Healthcheck') {
            steps {
                sleep(5)
                dir('e2e-tests'){
                    sh 'mvn verify -Dskipe.surefire.tests'
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-tests/target/surefire-reports/*.xml, e2e-tests/target/surefire-reports/*.xml, e2e-tests/target/failsafe-reports/*.xml'  
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', onlyIfSuccessful: true
        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build $BUILD_NUMBER has failed', to: 'pauloabsouto+jenkins@gmail.com'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build successful!', to: 'pauloabsouto+jenkins@gmail.com'
        }
    }
}
