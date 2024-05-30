def img
pipeline {
    agent any
    
    environment {
        VIRTUAL_ENV = "${WORKSPACE}/venv"
        PATH = "${VIRTUAL_ENV}/bin:${env.PATH}"
        registry = "daedov1/python-jenkins" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'DOCKERHUB'
        githubCredential = 'GITHUB'
        dockerImage = ''
    }

    stages {
        
        stage('checkout') {
            steps {
                git branch: 'main',
                credentialsId: githubCredential,
                url: 'https://github.com/daedov/practice-jenkins-medium.git'
            }
        }

        stage('Prepare Environment') {
            steps {
                script {
                    if (!fileExists('venv')) {
                        sh 'python3 -m venv venv'
                    }
                }
                sh 'source venv/bin/activate && pip install -r requirements.txt'
            }
        }
        
        stage ('Test'){
            steps {
                sh "source venv/bin/activate && pytest testRoutes.py"
            }
        }
        
        stage ('Clean Up'){
            steps{
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${env.JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'docker rm ${env.JOB_NAME}'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }

        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com ', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
                    
        stage('Deploy') {
           steps {
                sh label: '', script: "docker run -d --name ${env.JOB_NAME} -p 5000:5000 ${img}"
          }
        }

      }
    }