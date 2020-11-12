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
                    if [ -f "cloud2.pem" ]
                    then
                        echo "file exists..."
                    else
                        aws ec2 create-key-pair \
                          --region us-east-1 \
                          --key-name cloud2.pem \
                          --query KeyMaterial \
                          --output text > cloud2.pem
                        chmod 400 cloud2.pem
                        ssh-keygen -y -f cloud2.pem >> cloudpublic.pem
                    fi
                '''
            }
        }
		stage('create-cluster'){
            agent any
            steps{
                sh '''
                    #!/bin/sh
                    running=$(sudo lsof -i:80) || true
                    if [ "$running" != '' ]
                    then
                        docker-compose down
                        exist="$(aws eks list-clusters | grep jimmy-cluster2)" || true
                        if [ "$exist" == '' ]
                        then
                            eksctl create cluster \
                                --name jimmy-cluster2 \
                                --version 1.18 \
                                --region us-east-1 \
                                --nodegroup-name my-nodes \
                                --node-type t2.small \
                                --nodes 1 \
                                --nodes-min 1 \
                                --nodes-max 2 \
                                --ssh-access \
                                --ssh-public-key  cloudpublic.pem \
                                --managed
                        else
                            echo 'no need to create cluster...'
                        fi
                    else
                        echo 'app is not running with docker-compose up -d'
                    fi
                '''
            }
        }
    }
}