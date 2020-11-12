pipeline{
    agent any
    environment{
        MYSQL_DATABASE_HOST = "database-42.cbanmzptkrzf.us-east-1.rds.amazonaws.com"
        MYSQL_DATABASE_PASSWORD = "Clarusway"
        MYSQL_DATABASE_USER = "admin"
        MYSQL_DATABASE_DB = "phonebook"
        MYSQL_DATABASE_PORT = 3306
        PATH="/usr/local/bin/:${env.PATH}"
    }
    stages{
        stage("compile"){
           agent{
               docker{
                   image 'python:alpine'
               }
           }
           steps{
               withEnv(["HOME=${env.WORKSPACE}"]) {
                    sh 'pip install -r requirements.txt'
                    sh 'python -m py_compile src/*.py'
                }
           }
        }
        stage('test'){
            agent {
                docker {
                    image 'python:alpine'
                }
            }
            steps {
                withEnv(["HOME=${env.WORKSPACE}"]) {
                    sh 'python -m pytest -v --junit-xml results.xml src/appTest.py'
                }
            }
            post {
                always {
                    junit 'results.xml'
                }
            }
        }
        stage('build'){
            agent any
            steps{
                sh "docker build -t jenkins-lab/nov4 ."
                sh "docker tag jenkins-lab/nov4:latest 612215931034.dkr.ecr.us-east-1.amazonaws.com/jenkins-lab/nov4:latest"
            }
        }
        stage('push'){
            agent any
            steps{
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 612215931034.dkr.ecr.us-east-1.amazonaws.com"
                sh "docker push 612215931034.dkr.ecr.us-east-1.amazonaws.com/jenkins-lab/nov4:latest"
            }
        }
        stage('compose'){
            agent any
            steps{
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 612215931034.dkr.ecr.us-east-1.amazonaws.com"
                sh "docker-compose up -d"
            }
        }
		stage('get-keypair'){
            agent any
            steps{
                sh '''
                    if [ -f "cloud.pem" ]
                    then
                        echo "file exists..."
                    else
                        aws ec2 create-key-pair \
                          --region us-east-1 \
                          --key-name cloud.pem \
                          --query KeyMaterial \
                          --output text > cloud.pem
                        chmod 400 cloud.pem
                        ssh-keygen -y -f cloud.pem >> mattsJenkinsKey3_public.pem
                    fi
                '''
            }
        }
    }
}