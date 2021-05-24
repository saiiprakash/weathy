pipeline {
    agent any
    tools {
        dotnetsdk 'dotnet5'
        docker 'docker'
    }
    stages {
    	stage('Build') 	{
                steps {
                        sh 'dotnet publish src/weathy.csproj -c release'
                }
    	}
    	stage('Archival') {
                steps {
                        archiveArtifacts 'bin/release/net5.0/publish/*.dll'
                }
    	}
        stage('Test cases') {
                steps {
                        sh 'dotnet test test/weathy-test.csproj --logger "trx;LogFileName=UnitTests.trx"'
                        step([$class: 'MSTestPublisher', testResultsFile:"**/*.trx", failOnError: true, keepLongStdio: true])
                        mstest testResultsFile:"**/*.trx", keepLongStdio: true
                        //junit 'temporary-junit-reports/*.xml'
                }
        }
        stage('Build Image') {
            steps {
                sh '''
                    docker build --no-cache -t weathy-image:latest src
                    docker tag weathy-image:latest registry.heroku.com/weathy-image:v${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    heroku container:login
                    docker push registry.heroku.com/weathy-image:v${BUILD_NUMBER}
                '''
            }
        }
        stage('Deploy in Heroku') {
            steps {
                sh '''
                    heroku container:login
                    heroku container:release web --app weathy-app
                    docker-compose up -d
                '''
            }
        }
        stage('Test the deployment') {
            steps {
                sh '''
                    curl -X 'GET' 'http://localhost:5200/api/v1/weather'
                    curl -X 'GET' 'http://localhost:5000/api/v2/weather?city=london'
                '''
            }
        }
    }
    
    post {
        always {
            notify('started')
        }
        failure {
            notify('err')
        }
        success {
            notify('success')
        }
    }    
}

def notify(status){
    emailext (
    to: "devops.kphb@gmail.com",
    subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
    body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME}  [${env.BUILD_NUMBER}]</a></p>""",
    )
}
