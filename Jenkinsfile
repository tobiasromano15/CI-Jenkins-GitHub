pipeline {
    agent any

    environment {
        APP_VERSION = "1.0.$BUILD_ID"
        SNYK_TOKEN = credentials("snyk-api-token")
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Veverita-Engineering/Customers-API'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw --version'
                sh "./mvnw -DskipTests -Drevision=$APP_VERSION clean package"
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Parallel Stage') {
            failFast true
            parallel {
                stage('Test') {
                    steps {
                        sh "./mvnw test"
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }

                stage('SCA') {
                    steps {
                        script {
                          def RESULT = sh returnStatus: true, script: './mvnw snyk:test'
                          if ( RESULT == 1 ) {
                            unstable("There are SCA issues!")
                          }
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "labuser-ssh-key", keyFileVariable: 'temp_key')]) {
                    sh 'ssh -T -i $temp_key -o "StrictHostKeyChecking=no" localhost -p 2201'
                    sh 'scp -i $temp_key -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -P 2201 target/customers-$APP_VERSION.jar localhost:customers.jar'
                    sh '''
                        ssh -i $temp_key -o "StrictHostKeyChecking=no" localhost -p 2201 '
                        pkill -9 java
                        nohup java -jar -Djava.io.tmpdir=/home/labsuser/.tmp -Dserver.port=8081 customers.jar start >> spring.log 2>&1 &
                        '
                    '''
                }
            }
        }

        stage('API Tests') {
            steps {
                nodejs(nodeJSInstallationName: 'node-lts') {
                    retry(count: 3) {
                        sh 'node --version'
                        sh "newman run CustomersAPI.json --env-var 'baseUrl=http://localhost:8081' --env-var 'version=$APP_VERSION'"
                    }
                }
            }
        }
    }
}
